---
title: MyBatis-Plus使用总结
date: 2019-07-05 13:46:12
tags:
  - MyBatis-Plus
  - MyBatis
  - 总结
categories:
  - MyBatis
---

# MyBatis-Plus是什么？

MyBatis-Plus 是一个 MyBatis 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生

# 为什么有MyBatis-Plus？

一切的目的都是为了少写甚至不写 `Mapper.xml`，通过 Java 代码就能实现数据库的 CRUD

<!-- more -->

# 特性

- 无侵入、损耗小、强大的 CURD 操作
- 支持 Lambda 形式调用，支持多种数据库

# 准备数据库与数据

这里使用的是 MySQL 数据库

```sql
CREATE TABLE USER (
    id BIGINT PRIMARY KEY NOT NULL AUTO_INCREMENT COMMENT 'id',
    name VARCHAR(30) DEFAULT NULL COMMENT '姓名',
    age INT DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) DEFAULT NULL COMMENT '邮箱',
    manager_id BIGINT DEFAULT NULL COMMENT '直属上级id',
    create_time DATETIME DEFAULT NULL COMMENT '创建时间'
) ENGINE=INNODB CHARSET=UTF8;

INSERT INTO user (id, name, age, email, manager_id, create_time)
VALUES (1, '大boss', 40, 'boss@baomidu.com', null, '2019-01-11 14:20:20'),
(2, '王天风', 25, 'wtf@baomidu.com', 1, '2019-02-05 11:22:22'),
(3, '李艺伟', 28, 'lyw@baomidu.com', 1, '2019-02-14 08:31:16'),
(4, '张雨琪', 31, 'zjq@baomidu.com', 1, '2019-01-14 09:15:15'),
(5, '刘红雨', 32, 'lhm@baomidu.com', 1, '2019-01-14 09:48:16');
```

# 依赖

一个数据库驱动，版本由 spring-boot-starter-parent 指定

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

MyBatis-Plus 的 SpringBoot Starter

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>
```

简化 Getter、Setter 的 lombok

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

# 配置

## 数据库连接

打开 `application.yml`，加入以下配置

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mp?useSSL=false&serverTimezone=GMT%2B8
    username: root
    password:
```

## 日志

dao 所在的包名是 `com.ikutarian.mp.dao`，因此 `application.yml` 中指定 dao 日志的输出如下，注意需要使用 `trace` 级别的日志，这样可以看到比 `debug` 级别更多的日志信息

```yml
logging:
  level:
    root: warn
    com.ikutarian.mp.dao: trace
  pattern:
    console: '%p%m%n'
```

# 常用注解

1. @TableName：表名
2. @TableId：表的主键
3. @TableField：表的字段名

如果 Java Entity 的属性不是表里的字段，可以这样声明

```java
@TableField(exist = false)
```

# 创建 Entity

默认 Entity 的属性是驼峰命名的名称

```java
package com.ikutarian.mp.entity;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import java.util.Date;

@Getter
@Setter
@ToString
public class User {

    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    private Integer age;
    private String email;
    private Long managerId;
    private Date createTime;
}
```

# 创建 Dao

新建一个接口，继承 `BaseMapper`，并且传入 Entity。这样 Dao 接口写好了

```java
package com.ikutarian.mp.dao;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.ikutarian.mp.entity.User;

public interface UserMapper extends BaseMapper<User> {
}
```

现在就可以利用 MyBatis-Plus 提供的一系列方法了

# 新增

```java
@Test
public void insert() {
    User user = new User();
    user.setName("刘明强");
    user.setAge(31);
    user.setManagerId(1L);
    user.setCreateTime(new Date());

    int rows = userMapper.insert(user);
    System.out.println("影响记录数: " + rows);
    System.out.println("User的新主键是: " + user.getId());
}
```

控制台输出

```
DEBUG==>  Preparing: INSERT INTO user ( name, age, manager_id, create_time ) VALUES ( ?, ?, ?, ? ) 
DEBUG==> Parameters: 刘明强(String), 31(Integer), 1(Long), 2019-09-03 21:14:46.349(Timestamp)
DEBUG<==    Updates: 1
影响记录数: 1
User的新主键是: 6
```

# 查询

打开 `BaseMapper` 的源码文件，可以看到提供了如下几个查询方法

```java
/**
 * 根据 ID 查询
 *
 * @param id 主键ID
 */
T selectById(Serializable id);

/**
 * 查询（根据ID 批量查询）
 *
 * @param idList 主键ID列表(不能为 null 以及 empty)
 */
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);

/**
 * 查询（根据 columnMap 条件）
 *
 * @param columnMap 表字段 map 对象
 */
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);

/**
 * 根据 entity 条件，查询一条记录
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

/**
 * 根据 Wrapper 条件，查询总记录数
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

/**
 * 根据 entity 条件，查询全部记录
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

/**
 * 根据 Wrapper 条件，查询全部记录
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

/**
 * 根据 Wrapper 条件，查询全部记录
 * <p>注意： 只返回第一个字段的值</p>
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

/**
 * 根据 entity 条件，查询全部记录（并翻页）
 *
 * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

/**
 * 根据 Wrapper 条件，查询全部记录（并翻页）
 *
 * @param page         分页查询条件
 * @param queryWrapper 实体对象封装操作类
 */
IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

## 根据主键查询

MyBatis-Plus 提供了两个方法：

- `T selectById(id)`：根据 ID 查询
- `List<T> selectBatchIds(idList)`： 根据 ID 批量查询

比如

```java
/**
 * 根据 ID 查询
 */
@Test
public void selectById() {
    User user = userMapper.selectById(1L);

    System.out.println(user);
}

/**
 * 根据ID 批量查询
 */
@Test
public void selectBatchIds() {
    List<Long> ids = Arrays.asList(1L, 2L, 3L);
    List<User> users = userMapper.selectBatchIds(ids);

    users.forEach(System.out::println);
}
```

## 根据字段名与字段值查询

MyBatis-Plus 提供了：

- `selectByMap(Map<String, Object> columnMap)`

比如 SQL 如下

```sql
SELECT 
    *
FROM
    user
WHERE 
    name = '王天风' AND age = 25
```

Java 代码是

```java
/**
 * 根据 columnMap 条件
 */
@Test
public void selectByMap() {
    // 将查询条件包装成 Map<String, Object>
    Map<String, Object> columnMap = new HashMap<>();
    columnMap.put("name", "王天风");
    columnMap.put("age", 25);
    // 传入查询条件
    List<User> users = userMapper.selectByMap(columnMap);

    users.forEach(System.out::println);
}
```

## 条件构造器（Wrapper）查询

具体 API 看[官方文档](https://mp.baomidou.com/guide/wrapper.html)

打开 `com.baomidou.mybatisplus.core.conditions.AbstractWrapper` 这个类，提供了很多的条件构造方法。为了方便说明，现在已几个需求来进行说明

1. 名字中包含“雨”并且年龄小于40

SQL 是

```sql
SELECT
    *
FROM
    user 
WHERE
    name LIKE '%雨%'
    AND age < 40
```

Java 代码

```java
@Test
public void selectByWrapper() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .like("name", "雨")
            .lt("age", 40));

    users.forEach(System.out::println);
}
```

2. 名字中包含“雨”并且年龄大于等于20且小于40并且email不为空

SQL是

```sql
SELECT
    * 
FROM 
    user 
WHERE
    name LIKE '%雨%' 
    AND age >= 20 
    AND age <= 40 
    AND email IS NOT NULL
```

Java 代码

```java
@Test
public void selectByWrapper2() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .like("name", "雨")
            .between("age", 20, 40)
            .isNotNull("email"));

    users.forEach(System.out::println);
}
```

3. 姓王或者年龄大于等于25，按照年龄降序排列，年龄相同按照id升序排列

SQL是

```sql
SELECT
    *
FROM
    user
WHERE
    name LIKE '王%'
    OR age >= 25
ORDER BY
    age DESC,
    id ASC
```

Java 代码

```java
@Test
public void selectByWrapper3() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .likeRight("name", "王")
            .or()
            .ge("age", 40)
            .orderByDesc("age")
            .orderByAsc("id"));

    users.forEach(System.out::println);
}
```

4. 创建日期为2019年2月14日，并且直属上级姓王

SQL为

```sql
SELECT
    *
FROM
    user
WHERE
    date_forma(create_time, '%Y-%m-%d') = '2019-02-14'
    AND
    manager_id IN (SELECT id FROM user WHERE name LIKE '王%')
```

Java代码。这里要使用 [apply](https://mp.baomidou.com/guide/wrapper.html#apply) 和 [inSql](https://mp.baomidou.com/guide/wrapper.html#insql)

`apply` 用于调用 SQL 的函数。`inSql` 用在调用 `IN` 后面拼接 SQL 语句的场景

```java
@Test
public void selectByWrapper4() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .apply("date_format(create_time,'%Y-%m-%d') = {0}", "2019-02-14")
            .inSql("manager_id", "SELECT id FROM user WHERE name LIKE '王%'"));
    
    users.forEach(System.out::println);
}
```

**注意：**使用 `apply` 的时候推荐使用占位符 `{}`，这样可以防止 SQL 注入的风险

5. 姓王并且（年龄小于40或者邮箱不为空）

SQL是

```sql
SELECT
    *
FROM 
    user
WHERE
    name LIKE '王%'
    AND (age < 40 OR email IS NOT NULL)
```

这里 `AND` 后面跟着一个括号，括号里也是一个条件判断。这时候要用 [and](https://mp.baomidou.com/guide/wrapper.html#and)

Java代码

```java
@Test
public void selectByWrapper5() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .likeRight("name", "王")
            .and(qr -> qr.lt("age", 40)
                        .or()
                        .isNotNull("email")));
    
    users.forEach(System.out::println);
}
```

6. 姓王或者（年龄小于40并且大于20并且邮箱不为空）

SQL是

```sql
SELECT
    *
FROM
    user
WHERE
    name LIKE '王%'
    OR (age < 40 AND age > 20 AND email IS NOT NULL)
```

和上面的例子 5 一样，`OR` 后面也跟着一对括号。这时候可以用 [or](https://mp.baomidou.com/guide/wrapper.html#or)

Java代码

```java
@Test
public void selectByWrapper6() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .likeRight("name", "王")
            .or(qr -> qr.lt("age", 40)
                        .gt("age", 20)
                        .isNotNull("email")));
    
    users.forEach(System.out::println);
}
```

7. (年龄小于40或者邮箱不为空)并且姓王

SQL是

```sql
SELECT
    *
FROM
    user
WHERE
    (age < 40 OR email IS NOT NULL) AND name LIKE '王%'
```

Java代码

和例子 5 和例子 6 不同。这里是正常嵌套，SQL 句子前面不带 `AND` 或者 `OR`，这时候要借助 [nested](https://mp.baomidou.com/guide/wrapper.html#nested)

```java
@Test
public void selectByWrapper7() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .nested(qr -> qr.lt("age", 40)
                            .or()
                            .isNotNull("email"))
            .likeRight("name", "王"));

    users.forEach(System.out::println);
}
```

8. 年龄为 30 或者 31 或者 34 或者 35

SQL是

```sql
SELECT
    *
FROM
    user
WHERE
    age IN (30, 31, 34, 35)
```

Java代码

```java
@Test
public void selectByWrapper8() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .in("age", Arrays.asList(30, 31, 34, 35)));
    
    users.forEach(System.out::println);
}
```

9. 年龄为30、31、34、35，只返回满足条件的其中一条语句即可

SQL是

```sql
SELECT
    *
FROM
    user
WHERE
    age IN (30, 31, 34, 35)
LIMIT 1
```

Java代码

```java
@Test
public void selectByWrapper9() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .in("age", Arrays.asList(30, 31, 34, 35))
            .last("LIMIT 1"));
    
    users.forEach(System.out::println);
}
```

**注意：**`last()` 方法只能调用 1 次

## SELECT不列出全部字段

默认是查询所有字段，如果只需要特定的字段，要怎么做呢？，可以利用 [select](https://mp.baomidou.com/guide/wrapper.html#select) 来实现

10. 名字中包含“雨”并且年龄小于40，只列出 id 和 name 字段

SQL

```sql
SELECT
    id, name
FROM
    user 
WHERE
    name LIKE '%雨%'
    AND age < 40
```

Java代码

```java
@Test
public void selectByWrapper10() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>()
            .select("id", "name")
            .like("name", "雨")
            .lt("age", 40));
    
    users.forEach(System.out::println);
}
```

11. 名字中包含“雨”并且年龄小于40，除了 create_time 和 manager_id 字段外都列出

SQL

```sql
SELECT
    id, name, age, email 
FROM
    user 
WHERE
    name LIKE '%雨%'
    AND age < 40
```

Java代码

除了 create_time 和 manager_id 字段外都列出，`select` 也支持

```java
@Test
public void selectByWrapper11() {
    List<User> users = userMapper.selectList(new QueryWrapper<User>().like("name", "雨")
            .lt("age", 40)
            .select(User.class, fieldInfo -> !fieldInfo.getColumn().equals("create_time")
                    && !fieldInfo.getColumn().equals("manager_id"))); // select也可以写在后面
    
    users.forEach(System.out::println);
}
```

## condition的作用

有一些方法可以传入 condition 的参数，它有什么作用？举个例子来说明

比如这样的场景，如果 name 不为空才进行查询，email 不为空才进行查询

```java
@Test
public void selectByWrapperNoCondition() {
    String name = "王";
    String email = "";
    selectByWrapperNameEmail(name, email);
}

private void selectByWrapperNameEmail(String name, String email) {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();

    if (StringUtils.isNotEmpty(name)) {
        queryWrapper.like("name", name);
    }
    if (StringUtils.isNotEmpty(email)) {
        queryWrapper.like("email", email);
    }

    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

因为 name 不为空，email 为空，根据代码

```java
if (StringUtils.isNotEmpty(name)) {
    queryWrapper.like("name", name);
}
if (StringUtils.isNotEmpty(email)) {
    queryWrapper.like("email", email);
}
```

拼接得到的 SQL 为

```sql
SELECT * FROM user WHERE name LIKE '%王%'
```

虽然需求实现了，但是代码不够优雅。这时候，可以利用 3 个参数的 QueryWrapper 的方法进行 SQL 拼接

```java
@Test
public void selectByWrapperNoCondition() {
    String name = "王";
    String email = "asdf";
    selectByWrapperNameEmail2(name, email);
}

private void selectByWrapperNameEmail2(String name, String email) {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();

    queryWrapper.like(StringUtils.isNotEmpty(name), "name", name)
                .like(StringUtils.isNotEmpty(email), "email", email);

    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

原理就是：

> 只有第一个参数 `condition` 成立时才进行 SQL 拼接

## 创建条件构造器时传入实体对象

可以传一个实体对象，MyBatis-Plus 会根据实体对象的属性去创造 SQL。这个比 `selectByMap(Map<String, Object> columnMap)` 方法可读性更好

```java
@Test
public void selectByWrapperEntity() {
    User whereUser = new User();
    whereUser.setName("刘红雨");
    whereUser.setAge(32);
    QueryWrapper<User> queryWrapper = new QueryWrapper<>(whereUser);

    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

控制台输入日志为

```
DEBUG==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user WHERE name=? AND age=? 
DEBUG==> Parameters: 刘红雨(String), 32(Integer)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1094592041087729666, 刘红雨, 32, lhm@baomidu.com, 1088248166370832385, 2019-01-14 09:48:16
DEBUG<==      Total: 1
```

## allEq的使用

[allEq](https://mp.baomidou.com/guide/wrapper.html#alleq) 需要传入一个 `Map`。`allEq` 方法还可以接受第二个 boolean 类型的参数 `null2IsNull`。它表示如果传入的 KV 键值对中 V 是 `null` 的话，就转换成 `K IS NULL`

比如，这样的 SQL 语句

```sql
SELECT
    *
FROM 
    user 
WHERE 
    name = ? 
    AND age = ? 
    AND email IS NULL 
```

使用 `allEq` 实现的话，Java 代码如下

```java
@Test
public void selectByWrapperAllEq() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();

    Map<String, Object> params = new HashMap<>();
    params.put("name", "刘明强");
    params.put("age", 31);
    params.put("email", null);
    queryWrapper.allEq(params, true);

    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

如果传入的 `null2IsNull` 为 `false`，那么 KV 键值对中的 V 为 `null` 的话，就会被忽略

```java
@Test
public void selectByWrapperAllEq() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();

    Map<String, Object> params = new HashMap<>();
    params.put("name", "刘明强");
    params.put("age", 31);
    params.put("email", null);
    queryWrapper.allEq(params, false);

    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

日志打印的 SQL 语句是

```
DEBUG==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user WHERE name = ? AND age = ? 
DEBUG==> Parameters: 刘明强(String), 31(Integer)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1145231894878457857, 刘明强, 31, null, 1088248166370832385, 2019-06-30 15:25:36
DEBUG<==      Total: 1
```

可以看到 `email` - `null` 键值对被忽略了

## 将查询结果以 List<Map<String, Object>> 的形式返回

调用 `BaseMapper` 的 `selectMaps` 方法来实现

```java
/**
 * 根据 Wrapper 条件，查询全部记录
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

比如这个例子

```java
@Test
public void selectByWrapperMapList() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name", "雨")
            .lt("age", 40)
            .select("id", "name");

    List<Map<String, Object>> mapList = userMapper.selectMaps(queryWrapper);

    mapList.forEach(System.out::println);
}
```

有这样一个需求：按照直属上级分组，查询每组的平均年龄、最大年龄、最小年龄，并且取总年龄小于 500 的组。就可以利用 `BaseMapper.selectMaps` 取到结果，而不用去重新创建一个实体类来包装结果

SQL是

```sql
SELECT
    AVG(age) AS avg_age,
    MIN(age) AS min_age,
    MAX(age) AS max_age
FROM
    user
GROUP BY
    manager_id
HAVING
    SUM(age) < 500
```

Java代码

```java
@Test
public void selectByWrapperMapList2() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.select("AVG(age) AS avg_age", "MIN(age) AS min_age", "MAX(age) AS max_age")
                .groupBy("manager_id")
                .having("SUM(age) < {0}", 500);

    List<Map<String, Object>> mapList = userMapper.selectMaps(queryWrapper);

    mapList.forEach(System.out::println);
}
```

日志输出

```
DEBUG==>  Preparing: SELECT AVG(age) AS avg_age,MIN(age) AS min_age,MAX(age) AS max_age FROM user GROUP BY manager_id HAVING SUM(age) < ? 
DEBUG==> Parameters: 500(Integer)
TRACE<==    Columns: avg_age, min_age, max_age
TRACE<==        Row: 40.0000, 40, 40
TRACE<==        Row: 25.0000, 25, 25
TRACE<==        Row: 30.5000, 28, 32
DEBUG<==      Total: 3
{max_age=40, avg_age=40.0000, min_age=40}
{max_age=25, avg_age=25.0000, min_age=25}
{max_age=32, avg_age=30.5000, min_age=28}
```

## 只返回第一个字段的值

```java
/**
 * 根据 Wrapper 条件，查询全部记录
 * <p>注意： 只返回第一个字段的值</p>
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

比如，虽然使用 `select` 指定要返回 `id` 和 `name`，但数据只有 `id` 的数据

```java
@Test
public void selectByWrapperObjList() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name", "雨")
            .lt("age", 40)
            .select("id", "name");

    List<Object> objectList = userMapper.selectObjs(queryWrapper);

    objectList.forEach(System.out::println);
}
```

观察SQL日志，发现还是有查询 `name` 和 `age`，只不过只把第一个字段 `id` 的值取出返回

```
DEBUG==>  Preparing: SELECT id,name FROM user WHERE name LIKE ? AND age < ? 
DEBUG==> Parameters: %雨%(String), 40(Integer)
TRACE<==    Columns: id, name
TRACE<==        Row: 1094590409767661570, 张雨琪
TRACE<==        Row: 1094592041087729666, 刘红雨
DEBUG<==      Total: 2
1094590409767661570
1094592041087729666
```

## 获取符合条件的记录数

可以利用 `selectCount` 实现

```java
/**
 * 根据 Wrapper 条件，查询总记录数
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

一个例子

```java
@Test
public void selectByWrapperCount() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name", "雨")
            .lt("age", 40);

    Integer count = userMapper.selectCount(queryWrapper);

    System.out.println(count);
}
```

SQL日志是

```
DEBUG==>  Preparing: SELECT COUNT( 1 ) FROM user WHERE name LIKE ? AND age < ? 
DEBUG==> Parameters: %雨%(String), 40(Integer)
TRACE<==    Columns: COUNT( 1 )
TRACE<==        Row: 2
DEBUG<==      Total: 1
2
```

## 根据查询条件，只返回一条记录

利用 `selectOne` 实现

```java
/**
 * 根据 entity 条件，查询一条记录
 *
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

有个需要注意的地方：如果查询的结果不止 1 条就会抛出一个 `org.apache.ibatis.exceptions.TooManyResultsException` 异常。只有 0 条和 1 条时是正常的

比如

```java
@Test
public void selectByWrapperOne() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();

    queryWrapper.eq("name", "张雨琪")
            .eq("age", 31);        
    User user = userMapper.selectOne(queryWrapper);

    System.out.println(user);
}
```

## 自定义SQL查询时，使用 QueryWrapper

在自定义 SQL 时，也可以利用到 QueryWrapper 的便利性。有两种方式来实现：

1. 注解方式 Mapper.java

```java
@Select("select * from mysql_data ${ew.customSqlSegment}")
List<MysqlData> getAll(@Param(Constants.WRAPPER) Wrapper wrapper);
```

2. XML形式 Mapper.xml

```xml
<select id="getAll" resultType="MysqlData">
	SELECT * FROM mysql_data ${ew.customSqlSegment}
</select>
```

具体可以看[文档](https://mp.baomidou.com/guide/wrapper.html#%E4%BD%BF%E7%94%A8-wrapper-%E8%87%AA%E5%AE%9A%E4%B9%89sql)

### 第一种方式：注解方式 Mapper.java

```java
package com.ikutarian.mp.dao;

import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.toolkit.Constants;
import com.ikutarian.mp.entity.User;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import java.util.List;

public interface UserMapper extends BaseMapper<User> {
    
    // ${ew.customSqlSegment} 这是固定写法
    @Select("SELECT * FROM user ${ew.customSqlSegment}")
    List<User> selectAll(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
}
```

调用时

```java
@Test
public void selectByWrapperSQL() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name", "雨")
            .lt("age", 40);
    List<User> userList = userMapper.selectAll(queryWrapper);
    
    userList.forEach(System.out::println);
}
```

SQL日志

```
DEBUG==>  Preparing: SELECT * FROM user WHERE name LIKE ? AND age < ? 
DEBUG==> Parameters: %雨%(String), 40(Integer)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1094590409767661570, 张雨琪, 31, zjq@baomidu.com, 1088248166370832385, 2019-01-14 09:15:15
TRACE<==        Row: 1094592041087729666, 刘红雨, 32, lhm@baomidu.com, 1088248166370832385, 2019-01-14 09:48:16
DEBUG<==      Total: 2
User(id=1094590409767661570, name=张雨琪, age=31, email=zjq@baomidu.com, managerId=1088248166370832385, createTime=Mon Jan 14 09:15:15 CST 2019)
User(id=1094592041087729666, name=刘红雨, age=32, email=lhm@baomidu.com, managerId=1088248166370832385, createTime=Mon Jan 14 09:48:16 CST 2019)
```

### 第二种方式：XML形式 Mapper.xml

现在 SQL 语句不用注解了

```java
package com.ikutarian.mp.dao;

import com.baomidou.mybatisplus.core.conditions.Wrapper;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.baomidou.mybatisplus.core.toolkit.Constants;
import com.ikutarian.mp.entity.User;
import org.apache.ibatis.annotations.Param;
import java.util.List;

public interface UserMapper extends BaseMapper<User> {
        
    List<User> selectAll(@Param(Constants.WRAPPER) Wrapper<User> wrapper);
}
```

而是写在 user.xml 里

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ikutarian.mp.dao.UserMapper">

    <select id="selectAll2" resultType="com.ikutarian.mp.entity.User">
        SELECT * FROM user ${ew.customSqlSegment}
    </select>

</mapper>
```

使用还是一样

```java
@Test
public void selectByWrapperSQL2() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name", "雨")
            .lt("age", 40);
    List<User> userList = userMapper.selectAll2(queryWrapper);

    userList.forEach(System.out::println);
}
```

SQL日志输出也一样

```
DEBUG==>  Preparing: SELECT * FROM user WHERE name LIKE ? AND age < ? 
DEBUG==> Parameters: %雨%(String), 40(Integer)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1094590409767661570, 张雨琪, 31, zjq@baomidu.com, 1088248166370832385, 2019-01-14 09:15:15
TRACE<==        Row: 1094592041087729666, 刘红雨, 32, lhm@baomidu.com, 1088248166370832385, 2019-01-14 09:48:16
DEBUG<==      Total: 2
User(id=1094590409767661570, name=张雨琪, age=31, email=zjq@baomidu.com, managerId=1088248166370832385, createTime=Mon Jan 14 09:15:15 CST 2019)
User(id=1094592041087729666, name=刘红雨, age=32, email=lhm@baomidu.com, managerId=1088248166370832385, createTime=Mon Jan 14 09:48:16 CST 2019)
```

## 分页查询

要实现分页查询，需要两个步骤：

1. 配置分页插件
2. 调用 `BaseMapper` 的 `selectPage` 方法

### 配置分页插件

```java
package com.ikutarian.mp.config;

import com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```

### 调用 BaseMapper 的 selectPage 方法

```java
/**
 * 根据 entity 条件，查询全部记录（并翻页）
 *
 * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
 * @param queryWrapper 实体对象封装操作类（可以为 null）
 */
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

方法需要传入两个参数：

- `IPage<T>`
- `Wrapper<T>`

`IPage<T>` 可以使用实现类 `Page<T>`，`Wrapper<T>` 跟上面的一样使用 `QueryWrapper<T>` 即可

举个例子

```java
@Test
public void selectByWrapperPage() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    Page<User> page = new Page<>(1, 2);
    IPage<User> iPage = userMapper.selectPage(page, queryWrapper);

    System.out.println("总页数: " + iPage.getPages());
    System.out.println("总个数: " + iPage.getTotal());
    List<User> userList = iPage.getRecords();
    userList.forEach(System.out::println);
}
```

SQL日志是

```
DEBUG==>  Preparing: SELECT COUNT(1) FROM user WHERE age >= ? 
DEBUG==> Parameters: 26(Integer)
TRACE<==    Columns: COUNT(1)
TRACE<==        Row: 5
DEBUG==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user WHERE age >= ? LIMIT ?,? 
DEBUG==> Parameters: 26(Integer), 0(Long), 2(Long)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1087982257332887553, 大boss, 40, boss@baomidu.com, null, 2019-01-11 14:20:20
TRACE<==        Row: 1088250446457389085, 李艺伟, 28, lyw@baomidu.com, 1088248166370832385, 2019-02-14 08:31:16
DEBUG<==      Total: 2
总页数: 3
总个数: 5
User(id=1087982257332887553, name=大boss, age=40, email=boss@baomidu.com, managerId=null, createTime=Fri Jan 11 14:20:20 CST 2019)
User(id=1088250446457389085, name=李艺伟, age=28, email=lyw@baomidu.com, managerId=1088248166370832385, createTime=Thu Feb 14 08:31:16 CST 2019)
```

可以看到查询了两次，第一次先查询总记录数

```
DEBUG==>  Preparing: SELECT COUNT(1) FROM user WHERE age >= ? 
DEBUG==> Parameters: 26(Integer)
TRACE<==    Columns: COUNT(1)
TRACE<==        Row: 5
```

如果总记录数大于 0，再进行分页查询

```
DEBUG==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user WHERE age >= ? LIMIT ?,? 
DEBUG==> Parameters: 26(Integer), 0(Long), 2(Long)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1087982257332887553, 大boss, 40, boss@baomidu.com, null, 2019-01-11 14:20:20
TRACE<==        Row: 1088250446457389085, 李艺伟, 28, lyw@baomidu.com, 1088248166370832385, 2019-02-14 08:31:16
DEBUG<==      Total: 2
```

## 分页查询返回 List<Map<String, Object>> 结果

分页查询不止有 `selectPage` 方法，还有 `selectMapsPage` 方法。只不过 `selectMapsPage` 的返回值类型是 `IPage<Map<String, Object>>`

使用上面的例子，这次调用 `selectMapsPage` 方法

```java
@Test
public void selectByWrapperMapsPage() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    Page<User> page = new Page<>(1, 2);
    IPage<Map<String, Object>> mapIPage = userMapper.selectMapsPage(page, queryWrapper);

    System.out.println("总页数: " + mapIPage.getPages());
    System.out.println("总个数: " + mapIPage.getTotal());
    List<Map<String, Object>> mapList = mapIPage.getRecords();
    mapList.forEach(System.out::println);
}
```

SQL日志是

```
DEBUG==>  Preparing: SELECT COUNT(1) FROM user WHERE age >= ? 
DEBUG==> Parameters: 26(Integer)
TRACE<==    Columns: COUNT(1)
TRACE<==        Row: 5
DEBUG==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user WHERE age >= ? LIMIT ?,? 
DEBUG==> Parameters: 26(Integer), 0(Long), 2(Long)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1087982257332887553, 大boss, 40, boss@baomidu.com, null, 2019-01-11 14:20:20
TRACE<==        Row: 1088250446457389085, 李艺伟, 28, lyw@baomidu.com, 1088248166370832385, 2019-02-14 08:31:16
DEBUG<==      Total: 2
总页数: 3
总个数: 5
{create_time=2019-01-11 14:20:20.0, name=大boss, id=1087982257332887553, age=40, email=boss@baomidu.com}
{create_time=2019-02-14 08:31:16.0, manager_id=1088248166370832385, name=李艺伟, id=1088250446457389085, age=28, email=lyw@baomidu.com}
```

## 分页查询，但是不需要总记录数

如果要返回总记录数的话，分页查询需要发起 2 次查询。但是有些场景不需要返回总记录数，这时候再进行 2 次查询就是浪费资源了。这时候可以利用 `Page<T>` 的另外一个构造方法

```java
public Page(long current, long size, boolean isSearchCount) {
    this(current, size, 0, isSearchCount);
}
```

第三个参数 `isSearchCount` 表示是否进行 count 查询，传入 `false` 就不会进行 count 查询了

```java
@Test
public void selectByWrapperPageNoCount() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.ge("age", 26);
    Page<User> page = new Page<>(1, 2, false);  // 不进行count查询
    IPage<User> iPage = userMapper.selectPage(page, queryWrapper);

    System.out.println("总页数: " + iPage.getPages());
    System.out.println("总个数: " + iPage.getTotal());
    List<User> userList = iPage.getRecords();
    userList.forEach(System.out::println);
}
```

SQL日志是

```
DEBUG==>  Preparing: SELECT id,name,age,email,manager_id,create_time FROM user WHERE age >= ? LIMIT ?,? 
DEBUG==> Parameters: 26(Integer), 0(Long), 2(Long)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1087982257332887553, 大boss, 40, boss@baomidu.com, null, 2019-01-11 14:20:20
TRACE<==        Row: 1088250446457389085, 李艺伟, 28, lyw@baomidu.com, 1088248166370832385, 2019-02-14 08:31:16
DEBUG<==      Total: 2
总页数: 0
总个数: 0
User(id=1087982257332887553, name=大boss, age=40, email=boss@baomidu.com, managerId=null, createTime=Fri Jan 11 14:20:20 CST 2019)
User(id=1088250446457389085, name=李艺伟, age=28, email=lyw@baomidu.com, managerId=1088248166370832385, createTime=Thu Feb 14 08:31:16 CST 2019)
```

现在只有 1 次查询了

## 自定义SQL的分页查询

有两个要求：

- 返回值必须是 `IPage<T>`
- Mapper 接口的方法的第一个参数必须是 `Page<T>`

举个例子

user.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ikutarian.mp.dao.UserMapper">

    <select id="slectUserPage" resultType="com.ikutarian.mp.entity.User">
        SELECT * FROM user WHERE age > ${age}
    </select>

</mapper>
```

Java代码

```java
@Test
public void selectByWrapperPageNoCount() {
    Page<User> page = new Page<>(1, 2);
    IPage<User> iPage = userMapper.selectUserPage(page, 26);
    
    System.out.println("总页数: " + iPage.getPages());
    System.out.println("总个数: " + iPage.getTotal());
    List<User> mapList = iPage.getRecords();
    mapList.forEach(System.out::println);
}
```

SQL日志

```DEBUG==>  Preparing: SELECT COUNT(1) FROM user WHERE age > 26 
DEBUG==> Parameters: 
TRACE<==    Columns: COUNT(1)
TRACE<==        Row: 5
DEBUG==>  Preparing: SELECT * FROM user WHERE age > 26 LIMIT ?,? 
DEBUG==> Parameters: 0(Long), 2(Long)
TRACE<==    Columns: id, name, age, email, manager_id, create_time
TRACE<==        Row: 1087982257332887553, 大boss, 40, boss@baomidu.com, null, 2019-01-11 14:20:20
TRACE<==        Row: 1088250446457389085, 李艺伟, 28, lyw@baomidu.com, 1088248166370832385, 2019-02-14 08:31:16
DEBUG<==      Total: 2
总页数: 3
总个数: 5
User(id=1087982257332887553, name=大boss, age=40, email=boss@baomidu.com, managerId=null, createTime=Fri Jan 11 14:20:20 CST 2019)
User(id=1088250446457389085, name=李艺伟, age=28, email=lyw@baomidu.com, managerId=1088248166370832385, createTime=Thu Feb 14 08:31:16 CST 2019)
```

## lambda 条件构造器查询

上面使用 `QueryWrapper` 和 `UpdateWrapper` 的时候，都需要手工填写字段名。如果表结构改了，那么就要去手工查找与替换字段名。如果用 lambda 的条件构造器的话，就可以省略很多工作量了

现在使用 lambda 条件构造器重写例子 1 的代码。名字中包含“雨”并且年龄小于40，SQL 是

```sql
SELECT
    *
FROM
    user 
WHERE
    name LIKE '%雨%'
    AND age < 40
```

Java 代码

```java
@Test
public void selectLambda() {
    List<User> userList = userMapper.selectList(new LambdaQueryWrapper<User>()
            .like(User::getName, "雨")
            .lt(User::getAge, 40));

    userList.forEach(System.out::println);
}
```

# 更新

MyBatis-Plus 提供了两个关于更新的方法

```java
/**
 * 根据 ID 修改
 *
 * @param entity 实体对象
 */
int updateById(@Param(Constants.ENTITY) T entity);

/**
 * 根据 whereEntity 条件，更新记录
 *
 * @param entity        实体对象 (set 条件值,可以为 null)
 * @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
 */
int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
```

## 根据ID更新

Java代码

```java
@Test
public void updateById() {
    User user = new User();
    user.setId(1088248166370832385L);
    user.setAge(26);
    user.setEmail("wtf2@ikutarian.com");

    int rows = userMapper.updateById(user);  // 返回影响的记录数
    System.out.println("影响记录数: " + rows);
}
```

SQL日志

```
DEBUG==>  Preparing: UPDATE user SET age=?, email=? WHERE id=? 
DEBUG==> Parameters: 26(Integer), wtf2@ikutarian.com(String), 1088248166370832385(Long)
DEBUG<==    Updates: 1
影响记录数: 1
```

## 以条件构造器为参数的更新

需要传入两个参数：

1. 实体
2. 条件构造器：`UpdateWrapper<T>`

实体就是 SQL 语句中的 `SET xx = xx` 的内容，条件构造器就是 SQL 中 `where xxx` 的部分

比如这样一条 SQL

```
UPDATE
    user
SET
    email = ''
    AND age = 29
WHERE
    name = '李艺伟'
    AND age = 28
```

用 Java 代码实现就是

```java
@Test
public void updateByWrapper() {
    // SET age=?, email=?
    User user = new User();
    user.setEmail("lyw2019@outlook.com");
    user.setAge(29);

    // WHERE name = ? AND age = ? 
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    updateWrapper.eq("name", "李艺伟")
            .eq("age", 28);

    int rows = userMapper.update(user, updateWrapper);
    System.out.println("影响记录数: " + rows);
}
```

SQL 日志是

```
DEBUG==>  Preparing: UPDATE user SET age=?, email=? WHERE name = ? AND age = ? 
DEBUG==> Parameters: 29(Integer), lyw2019@outlook.com(String), 李艺伟(String), 28(Integer)
DEBUG<==    Updates: 1
影响记录数: 1
```

## 以条件构造器为参数的更新 2.0

还有一种方法可以省略实体，直接使用 `UpdateWrapper<T>` 设置要更新的值

还是上面的例子，不过这次不需要传入实体，直接调用 `UpdateWrapper<T>` 的 `set` 方法来设置新的值

```java
@Test
public void updateByWrapper2() {
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    updateWrapper.eq("name", "李艺伟")  // WHERE name = ? AND age = ?
            .eq("age", 29)
            .set("email", "lyw@ikutarian.com")  //  // SET age=?, email=?
            .set("age", 30);
    int rows = userMapper.update(null, updateWrapper);  // 实体传 null，只需要 UpdateWrapper

    System.out.println("影响记录数: " + rows);
}
```

SQL日志

```
DEBUG==>  Preparing: UPDATE user SET email=?,age=? WHERE name = ? AND age = ? 
DEBUG==> Parameters: lyw@ikutarian.com(String), 30(Integer), 李艺伟(String), 29(Integer)
DEBUG<==    Updates: 1
影响记录数: 1
```

# 删除

MyBatis-Plus 提供了以下几个删除的 API

```java
/**
 * 根据 ID 删除
 *
 * @param id 主键ID
 */
int deleteById(Serializable id);

/**
 * 删除（根据ID 批量删除）
 *
 * @param idList 主键ID列表(不能为 null 以及 empty)
 */
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);

/**
 * 根据 columnMap 条件，删除记录
 *
 * @param columnMap 表字段 map 对象
 */
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);

/**
 * 根据 entity 条件，删除记录
 *
 * @param wrapper 实体对象封装操作类（可以为 null）
 */
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
```

## 根据ID删除

传入主键即可

```java
@Test
public void deleteById() {
    int rows = userMapper.deleteById(123L);
    System.out.println("影响记录数: " + rows);
}
```

## 根据ID批量删除

```java
@Test
public void deleteByIds() {
    int rows = userMapper.deleteBatchIds(Arrays.asList(1, 2, 3));
    System.out.println("影响记录数: " + rows);
}
```

## 根据columnMap条件，删除记录

`columnMap` 中存储的 KV 键值对，就是 SQL 中对应的 `WHERE k = v` 部分

```java
@Test
public void deleteByMap() {
    Map<String, Object> columnMap = new HashMap<>();
    columnMap.put("name", "test");
    columnMap.put("age", 25);

    int rows = userMapper.deleteByMap(columnMap);
    System.out.println("影响记录数: " + rows);
}
```

## 使用条件构造器进行删除

使用 `QueryWrapper<T>` 来实现 DELETE SQL 语句中 `WHERE xx = xx` 的部分

```java
@Test
public void deleteByWrapper() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.eq("name", "test")
            .eq("age", 25);

    int rows = userMapper.delete(queryWrapper);
    System.out.println("影响记录数: " + rows);
}
```

SQL日志是

```
DEBUG==>  Preparing: DELETE FROM user WHERE name = ? AND age = ? 
DEBUG==> Parameters: test(String), 25(Integer)
DEBUG<==    Updates: 0
影响记录数: 0
```