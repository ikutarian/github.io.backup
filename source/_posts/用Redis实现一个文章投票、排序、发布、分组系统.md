---
title: 用Redis实现一个文章投票、排序、发布、分组系统
date: 2019-04-15 10:09:48
tags:
  - Redis
  - hash
  - set
  - zset
  - 投票
  - 排序
  - 分组
categories:
  - Redis
---

一个文章发布网站，可以对文章进行一些操作：

1. 发布
2. 投票
3. 按照时间排序得到文章列表
4. 按照投票进行排序得到文章列表
4. 分组

<!-- more -->

文章可以投票，规则是：如果一篇文章有 200 个以上的赞，就是一个好文章；并且网站首页每天会列出最新的 50 篇的好文章

为了能得到一个随着时间流逝而不断减少的评分，文章的评分标准就有 2 个维度：赞的个数、发布时间。具体的计算方法为：将文章的赞的数目乘以一个常量，然后加上文章的发布时间，得出的结果就是文章的评分

```
评分 = 赞的个数 * 常量 + 发布时间
```

使用时间戳来计算文章的发布时间，常量设置为 432。这个常量是将 1 天的秒数（60 * 60 * 24 = 86400）除以好文章需要的赞的数目（200）得到的。文章每获取一个赞，程序就需要将文章的评分加上 432 分。并且只有发布时间之后的一周内才可以进行评分，超时就不能评分了

这种需求可以利用 Redis 来实现

# Redis是什么

速度非常快的非关系型数据库，可以存储键（key）与 5 种不同类型的值（value）之间的映射（mapping）

# 类型

键（key）都是字符串，值（value）的 5 种类型的值如下：

1. 字符串
2. 列表
3. 哈希
4. 集合
5. 有序集合

所以映射关系如下：

1. string: string
2. string: list
3. string: hash
4. string: set
5. string: zset

# 使用的数据结构

## 文章信息

使用 hash 来存储文章信息。比如一个 ID 为 `92617` 的文章，如图

{% asset_img Snipaste_2019-04-13_21-23-01.png %}

操作指令如下

```
127.0.0.1:6379> hset article:92617 title "Go to statement considered harmful"
(integer) 1
127.0.0.1:6379> hset article:92617 link "http://goo.gl/kZUSu"
(integer) 1
127.0.0.1:6379> hset article:92617 poster user:83271
(integer) 1
127.0.0.1:6379> hset article:92617 time 1331382699.33
(integer) 1
127.0.0.1:6379> hset article:92617 votes 528
(integer) 1
127.0.0.1:6379> hgetall article:92617
 1) "title"
 2) "Go to statement considered harmful"
 3) "link"
 4) "http://goo.gl/kZUSu"
 5) "poster"
 6) "user:83271"
 7) "time"
 8) "1331382699.33"
 9) "votes"
10) "528"
```

## 文章发布时间

用 zset 存储。成员（member）为文章 ID，分值（score）为文章的发布时间

{% asset_img Snipaste_2019-04-13_21-26-51.png %}

操作指令如下

```
zadd time 1332065417 article:100408
zadd time 1332075503 article:100635
zadd time 1332082035 article:100716
```

## 文章评分

用 zset 存储。成员（member）为文章 ID，分值（score）为文章的评分

{% asset_img Snipaste_2019-04-13_21-37-06.png %}

操作指令如下

```
zadd time 1332164063 article:100635
zadd time 1332174713 article:100408
zadd time 1332225027 article:100716
```

## 文章投票

为了防止用户对同一篇文章进行多次投票，还需要为每篇文章记录记录一个已投票用户名单，可以使用 set 来存储

{% asset_img Snipaste_2019-04-13_21-40-01.png %}

并且，文章发布一周之后就不能再次投票了，记录文章已投票用户名单的集合也将被删除

```
sadd voted:100408 user:234487
```

# 实现

## 文章投票

当用户尝试对一篇文章进行投票时，程序使用 `ZSCORE` 命令查询文章发布时间的有序集合，判断文章发布的时间是否超过一周。如果还可以投票，使用 `SADD` 把用户添加到该文章的已投票用户名单的集合里。如果添加成功的话，说明用户是第一次对这篇文章进行投票，于是使用 `ZINCRBY` 给文章的评分增加 432 分，并使用 `HINCRBY` 命令对文章信息 hash 里的投票数目进行更新

```java
private static final int ONE_WEEK_IN_SECONDS = 7 * 86400;
private static final int VOTE_SCORE = 432;

public String articleVote(Jedis jedis, String userId, String articleId) {
    long cutoff = System.currentTimeMillis() - ONE_WEEK_IN_SECONDS;
    if (jedis.zscore("time", "article:" + articleId) < cutoff) {
        return "距文章发布时间超过一周，不允许投票";
    }

    if (jedis.sadd("voted:" + articleId, userId) != 1) {
        return "该用户已经对文章投过票了";
    }

    jedis.zincrby("score", VOTE_SCORE, "article:" + articleId);
    jedis.hincrBy("article", "votes", 1);
    
    return "投票成功";
}
```

## 发布文章

发布一篇文章需要创建一个新的文章 ID，可以通过对一个计数器执行 `INCR` 命令来完成。接着程序使用 `SADD` 把文章作者的 ID 加入到已投票用户名单的集合里，并且使用 `EXPIRE` 命令为这个集合设置一个过期时间，Redis 会在一周之后自动删除这个集合。然后，程序使用 `HMSET` 命令存储文章信息，并使用两个 `SADD` 命令将文章的初始评分和发布时间分别添加到两个有序集合里

```java
/**
 * 发布文章
 *
 * @param userId 用户ID
 * @param title 文章标题
 * @param link 文章链接
 */
public Long postArticle(Jedis jedis, String userId, String title, String link) {
    Long newArticleId = jedis.incr("article");

    jedis.sadd("voted:" + newArticleId, userId);
    jedis.expire("voted:" + newArticleId, ONE_WEEK_IN_SECONDS);

    long now = System.currentTimeMillis();

    Map<String, String> articleMap = new HashMap<>();
    articleMap.put("title", title);
    articleMap.put("link", link);
    articleMap.put("poster", "user:" + userId);
    articleMap.put("time", String.valueOf(now);
    articleMap.put("votes", String.valueOf(1));
    jedis.hmset("article:" + newArticleId, articleMap);

    jedis.zadd("score", now + VOTE_SCORE, "article:" + newArticleId);
    jedis.zadd("time", now, "article:" + newArticleId);
    
    return newArticleId;
}
```

## 文章列表

如何按照评分从高到低以及按照时间从新到旧的顺序得到文章列表？可以使用 `ZREVRANGE` 命令从时间有序列表或者评分有序列表取出多个文章 ID，然后再调用 `HGETALL` 命令从文章信息的 hash 里取出文章详情

因为默认是从小到大，所以才使用 `ZREVRANGE` 命令反过来从大到小

```java
private static final int ARTICLES_PER_PAGE = 25;

/**
 * 文章列表
 *
 * @param page  页码
 * @param order 排序，有 time 和 score 两种
 */
public List<Map<String, String>> getArticles(Jedis jedis, int page, String order) {
    List<Map<String, String>> articles = new ArrayList<>();

    int start = (page - 1) * ARTICLES_PER_PAGE;
    int end = start + ARTICLES_PER_PAGE - 1;

    Set<String> articleIds = jedis.zrevrange(order, start, end);
    for (String articleId : articleIds) {
        Map<String, String> articleMap = jedis.hgetAll("article:" + articleId);
        articleMap.put("id", articleId);

        articles.add(articleMap);
    }

    return articles;
}
```

## 文章分组

群组功能由两个部分组成，一个部分负责记录文章属于哪个分组，另一个负责取出群组里的文章。问了记录各个群组都存了那些文章，需要为每个群组创建一个 set，将同属一个群组的文章的 ID 放到集合里

下面是一个把文章加入群组和移出群组的方法

```java
/**
 * 把文章加入群组和移出群组
 *
 * @param articleId 文章ID
 * @param toAdd     要加入的群组
 * @param toRemove  要移出的群组
 */
public void addRemoveGroups(Jedis jedis, String articleId, List<String> toAdd, List<String> toRemove) {
    String article = "article:" + articleId;

    for (String groupName : toAdd) {
        jedis.sadd("group:" + groupName, article);
    }
    
    for (String groupName : toRemove) {
        jedis.srem("group:" + groupName, article);
    }
}
```

## 文章分组内的文章进行排序

群组内的文章也需要按照时间或者分数来进行排序，`ZINTERSTORE` 命令可以接受多个集合和多个有序集合作为输入，做一个交集，找出所有同时存在于集合和有序集合中的成员

比如一个“编程”分组里的文章，对它按照分值进行排序

{% asset_img Snipaste_2019-04-15_09-50-44.png %}

如果群组里的文章非常多，执行 `ZINTERSTORE` 命令就会比较花时间，为了减少资源占用，程序将 `ZINTERSTORE` 命令的计算结果缓存 60 秒

```java
public List<Map<String, String>> getGroupArticles(Jedis jedis, String groupName, int page, String order) {
    String key = order + ":" + groupName; // 群组计算结果的 key
    if (!jedis.exists(key)) {
        jedis.zinterstore(key, "groups:" + groupName, order);  // 取交集
        jedis.expire(key, 60);  // 缓存 60 秒
    }

    return getArticles(jedis, page, key);  // 获取文章列表
}
```

# 总结

熟悉 Redis 的 5 种值类型：

1. 字符串
2. 列表
3. 哈希
4. 集合
5. 有序集合

同时熟悉一些相关的操作命令，这样的问题就迎刃而解了