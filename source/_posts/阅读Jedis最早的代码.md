---
title: 阅读Jedis最早的代码
date: 2019-01-29 16:36:41
tags:
  - Jedis
  - Redis
  - 源码
categories:
  - 源码阅读
---

Jedis 最新版的代码太多了，阅读起来很吃力，从早期版本开始阅读的话会简单很多

<!-- more -->

# 怎么找老版本的代码？

打开 Jedis 的 [Github主页](https://github.com/xetorthio/jedis)，可以看到 commit 的记录个数

{% asset_img Snipaste_2019-01-29_15-34-53.png %}

点击 [commit](https://github.com/xetorthio/jedis/commits/master) 进去，拉到最低部，点击 `older` 按钮

{% asset_img Snipaste_2019-01-29_15-36-50.png %}

可以看到浏览器的地址栏目变化

{% asset_img Snipaste_2019-01-29_15-38-11.png %}

后面的数字就是 offset，目前的提交记录个数是 1535，那就试试 offset 为 1500。在浏览器中输入 https://github.com/xetorthio/jedis/commits/master?after=bf212393a11e449c064947124e0f244ccfc38bff+1500

这样就翻到了最后一页

{% asset_img Snipaste_2019-01-29_15-39-57.png %}

点击这个就可以进入当时的提交版本

{% asset_img Snipaste_2019-01-29_15-41-16.png %}

再点击 `Browse files` 就可以查看当时所有的代码文件了

{% asset_img Snipaste_2019-01-29_15-41-56.png %}

在旁边还可以看到所有的 tag

{% asset_img Snipaste_2019-01-29_15-43-47.png %}

点击就可以查看打上 tag 的代码了

# 第一个版本的代码

现在来分析一下签名为 `829e6d9862df0b17931afefb961021fd5ee89db8` 的代码

{% asset_img Snipaste_2019-01-29_16-43-11.png %}

我本地已经下载好了一份[代码](/uploads/jedis/jedis-829e6d9862df0b17931afefb961021fd5ee89db8.zip)

# 通信协议

要研究 Jedis 就需要一份 Redis 的通信协议：

- [中文版](http://doc.redisfans.com/topic/protocol.html)
- [英文版](https://redis.io/topics/protocol)

本地也下载好了一份 PDF 版的[文件](/uploads/jedis/Redis_protocol.pdf)

## 请求协议

协议的一般形式：

```
*<参数数量> CR LF
$<参数 1 的字节数量> CR LF
<参数 1 的数据> CR LF
...
$<参数 N 的字节数量> CR LF
<参数 N 的数据> CR LF
```

比如发送一条这样的命令：`SET a b`，请求内容为

```
*3
$3
SET
$1
a
$1
b
```

## 响应协议

一共有 5 种：

- status reply
- error reply
- integer reply
- bulk reply
- multi bulk reply

具体的格式看下图

{% asset_img 8793587-3becf73b1945c422.png %}