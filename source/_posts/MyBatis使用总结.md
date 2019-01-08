---
title: MyBatis使用总结
date: 2018-10-29 16:36:55
tags:
  - MyBatis
categories:
  - 数据库
---

总结一下我在学习与使用MyBatis的知识，以免忘记，也算是对学习过程的总结

<!-- more -->

## 一个能跑起来的例子

加入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

建立一张表，并插入数据

```sql
CREATE TABLE `country` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(30) DEFAULT NULL,
  `code` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO `country` VALUES 
('1', '中国', 'CN'),
('2', '美国', 'US'),
('3', '俄罗斯', 'RU'),
('4', '英国', 'GB'),
('5', '法国', 'FR');
```

实体类

**注意**：

> 实体类中要使用包装类型，不要使用基本数据类型，**这一点很重要**

```java
public class Country {

    private Long id;
    private String name;
    private String code;

    // getter
    // setter
    // toString
}
```

Mapper 接口。MyBatis 使用 Java 的动态代理，直接通过 Mapper 接口调用相应的方法，不需要接口实现类

```java
public interface CountryMapper {

    List<Country> selectAll();
}
```

Mapper 配置，放在 `resources/mapper/CountryMapper.xml`

`<mapper>` 标签的 `namespace` 指向 Mapper 接口的全名。`<select>` 的 `id` 属性指向 Mapper 接口的方法名

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ikutarian.mybatis.mapper.CountryMapper">

    <select id="selectAll" resultType="Country">
        select * from country
    </select>

</mapper>
```

数据库连接配置，放在 `resources/jdbc.properties`

```
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/mybatis?characterEncoding=utf-8
username=root
password=
```

Mybatis的配置，放在 `resources/mybatis-config.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <properties resource="jdbc.properties"/>

    <typeAliases>
        <package name="com.ikutarian.mybatis.model"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mapper/CountryMapper.xml"/>
    </mappers>

</configuration>
```

测试类

```java
public class Test {

    public static void main(String[] args) throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

        SqlSession session = sqlSessionFactory.openSession();
        try {
            CountryMapper countryMapper = session.getMapper(CountryMapper.class);
            List<Country> countryList = countryMapper.selectAll();
            for (Country country : countryList) {
                System.out.println(country);
            }
        } finally {
            session.close();
        }
    }
}
```

## 一个BRAC系统的所需要的表

先建立一个BRAC系统的表为例子，有以下几张表：

```sql
/* 用户表 */
CREATE TABLE user (
    id bigint NOT NULL AUTO_INCREMENT,
    username varchar(50),
    password varchar(50),
    email varchar(50),
    create_time datetime,
    update_time datetime,
    PRIMARY KEY(id)
);

INSERT INTO user VALUES 
('1', 'admin', '123456', 'admin@mybatis.tk', now(), now()),
('1001', 'test', '123456', 'test@mybatis.tk', now(), now());


/* 角色表 */
CREATE TABLE role (
    id bigint NOT NULL AUTO_INCREMENT,
    name varchar(50),    
    create_time datetime,
    update_time datetime,
    PRIMARY KEY(id)
);

INSERT INTO role VALUES 
('1', '管理员', now(), now()),
('2', '普通用户', now(), now());


/* 用户角色关联表 */
CREATE TABLE user_role (
    id bigint NOT NULL AUTO_INCREMENT,
    user_id bigint,    
    role_id bigint,    
    create_time datetime,
    update_time datetime,
    PRIMARY KEY(id)
);

INSERT INTO user_role VALUES 
('1', '1', '1', now(), now()),
('2', '1001', '2', now(), now());


/* 权限表 */
CREATE TABLE privilege (
    id bigint NOT NULL AUTO_INCREMENT,
    name varchar(50),    
    url varchar(200),    
    create_time datetime,
    update_time datetime,
    PRIMARY KEY(id)
);

INSERT INTO privilege VALUES 
('1', '用户管理', '/users', now(), now()),
('2', '角色管理', '/roles', now(), now()),
('3', '系统日志', '/logs', now(), now()),
('4', '人员维护', '/persons', now(), now()),
('5', '单位维护', '/companies', now(), now());


/* 角色权限关联表 */
CREATE TABLE role_privilege (
    id bigint NOT NULL AUTO_INCREMENT,
    role_id bigint,    
    privilege_id bigint,    
    create_time datetime,
    update_time datetime,
    PRIMARY KEY(id)
);

INSERT INTO role_privilege VALUES 
('1', '1', '1', now(), now()),
('2', '1', '3', now(), now()),
('3', '1', '2', now(), now()),
('4', '2', '4', now(), now()),
('5', '2', '5', now(), now());
```

## 重点知识

MyBatis 的重点知识，我觉得有以下几个：

- 输入映射
- 输出映射
- 动态SQL
- 关联查询

### 动态SQL —— WHERE

一句话说明：

> 用在 where 子句中，负责删除 and 和 or 关键词

有一个产品搜索功能，传入 productId 和 productName 进行搜索。根据 productId 和 productName 是否有值，SQL 语句会有这几种情况

```sql
SELECT * FROM product
SELECT * FROM product WHERE id = #{productId}
SELECT * FROM product WHERE name like CONCAT('%', trim(#{productName}), '%')
SELECT * FROM product WHERE id = #{productId} andname like CONCAT('%', trim(#{productName}), '%')
```

这种情况的话，可以用 `<where>` 和 `<if>` 标签

```xml
select
    <include refid="Base_Column_List"/>
from mmall_product
    <where>
        <if test="@com.ikutarian.mmall.util.MybatisUtils@isNotBlank(productName)">
            name like CONCAT('%', trim(#{productName}), '%')
        </if>
        <if test="productId != null">
            and id = #{productId}
        </if>
    </where>
```

`<where>`标签遇到 SQL 拼接结果是 `where and xxx` 或者 `where or xxx` 的话，会自动将 `and` 或者是 `or` 给忽略掉

注意，这里的 `and` 不能省略，否则 `<where>` 标签不知道是要添加 and 还是 or

```
<if test="productId != null">
    and id = #{productId}
</if>
```

### 动态SQL —— SET

一句话说明：

> 用在 set 子句中，负责删除查询语句末尾的逗号

```sql
update
    mmall_product
<set>
    <if test="name != null">
        name = #{name},
    </if>
    <if test="price != null">
        price = #{price},
    </if>
</set>
where id = #{id}
```

根据 name 与 price 是否有值，有几种情况

```sql
update mmall_product set name = 'xxx' where id = xxx;
update mmall_product set name = 'xxx' and price = xxx where id = xxx;
```

`<set>` 标签可以帮忙把 `price = #{price},` 和 `name = #{name},` 末尾的逗号去掉