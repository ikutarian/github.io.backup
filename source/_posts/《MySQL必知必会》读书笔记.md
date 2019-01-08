---
title: 《MySQL必知必会》读书笔记
date: 2018-12-03 15:28:46
tags:
  - MySQL
categories:
  - 读书笔记
---

重新系统地学习一遍 MySQL 的基础知识

<!-- more -->

# 第 19 章 插入数据

- 插入完整的行
- 插入行的一部分
- 插入多行
- 插入某些查询的结果

# 第 20 章 更新和删除数据

## 原则

1. 如果不是打算更新或者删除全表的话，不要忘了 `WHERE` 子句
2. 保证每个表都有主键
3. 在执行 `UPDATE` 或者 `DELETE` 之前，先用 `SELECT` 进行测试

# 第 21 章 创建和操纵表

## 创建表

### NULL 值

在创建表时，可以指定列是否可以为空。比如

```sql
CREATE TABLE user (
    name varchar(10) NOT NULL,
    address varchar(50)
);
```

`NOT NULL` 说明改列不能为空，如果没有写 `NOT NULL`，默认就是 `NULL`，表示可以为空

### 主键

主键可以使用单主键和联合主键

以供应商 id 为主键

```sql
CREATE TABLE vendors
(
  vend_id      int      NOT NULL AUTO_INCREMENT,
  vend_name    char(50) NOT NULL ,
  vend_address char(50) NULL
  PRIMARY KEY (vend_id)
) ENGINE=InnoDB;
```

订单详情表，订单号（`order_num`）和订单项 id（`order_item`）一起为联合主键

```sql
CREATE TABLE orderitems
(
  order_num  int          NOT NULL ,
  order_item int          NOT NULL ,
  prod_id    char(10)     NOT NULL ,
  quantity   int          NOT NULL ,
  item_price decimal(8,2) NOT NULL ,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;
```

### AUTO_INCREMENT

一张表只能有一个 `AUTO_INCREMENT`，并且它必须被索引（比如成为主键）

### 指定默认值

使用 `DEFAULT`，并且只支持常量不支持函数

### 引擎类型

使用 `ENGINE` 指定。几个常见的引擎：

- InnoDB 是一个可靠的事务处理引擎，不支持全文搜索
- MEMORY 在功能上等同于 MyISAM。但由于数据存储在内存中，速度很快（特别适合于临时表）
- MyISAM 是一个性能极高的引擎，支持全文搜索，但不支持事务处理

注意：

1. 引擎类型可以混用。比如这张表用 InnoDB 引擎，另外一张表用 MyISAM 引擎
2. 外键不能跨引擎

## 更新表

```sql
ALTER TABLE
```

## 删除表

```sql
DROP TABLE
```

## 重命名表

```sql
RENAME TABLE ... TO ...
```

# 第 26 章 管理事务处理

## 作用

维护数据库的完整性

## 引擎

并非所有的数据库引擎都支持事务

- MyISAM 不支持事务
- InnoDB 支持事务

## 专有名词

- 事务 transation
- 提交 commit
- 回滚 rollback
- 保存点 savepoint

## 哪些语句可以 rollback？

只有这三种语句可以回滚

- INSERT
- UPDATE
- DELETE

`CREATE` 和 `DROP` 没办法回滚

# 第 27 章 全球化和本地化

## 专有名词

- 字符集：CHARACTER SET
- 校对：COLLATION

## 为什么需要 COLLATION

进行排序的时候需要用到 COLLATION。比如按照字母顺序、大小写敏感等进行排序

## 查看支持的字符集和校对

查看支持的字符集

```sql
SHOW CHARACTER SET;
```

查看支持的校对

```sql
SHOW COLLATION;
```

有的字符集有多种校对，比如 `latin1` 就有很多种校对（`_cs` 表示区分大小写，`_ci` 表示不区分大小写）

```
+--------------------------+----------+-----+---------+----------+---------+
| Collation                | Charset  | Id  | Default | Compiled | Sortlen |
+--------------------------+----------+-----+---------+----------+---------+
| latin1_german1_ci        | latin1   |   5 |         | Yes      |       1 |
| latin1_swedish_ci        | latin1   |   8 | Yes     | Yes      |       1 |
| latin1_danish_ci         | latin1   |  15 |         | Yes      |       1 |
| latin1_german2_ci        | latin1   |  31 |         | Yes      |       2 |
| latin1_bin               | latin1   |  47 |         | Yes      |       1 |
| latin1_general_ci        | latin1   |  48 |         | Yes      |       1 |
| latin1_general_cs        | latin1   |  49 |         | Yes      |       1 |
| latin1_spanish_ci        | latin1   |  94 |         | Yes      |       1 |
```

## 查看默认的字符集和校对

```sql
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';
```

## 指定字符集和校对

0. 建数据库时指定

```sql
CREATE DATABASE mmall DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

1. 建表时指定

```sql
CREATE TABLE mytable (
  column1 INT,
  column2 VARCHAR(10)
) DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

2. 对列指定

```sql
CREATE TABLE mytable (
  column1 INT,
  column2 VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_general_ci
) DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

3. 在 SELECT 语句中某一列指定一个备用的 `COLLATE`

```sql
SELECT * FROM customers
ORDER BY lastname, firstname COLLATE latin1_general_cs;
```

对于 `CHARACTER SET` 和 `COLLATE` 有以下规则：

- 如果同时指定了 `CHARACTER SET` 和 `COLLATE`，就用这两者
- 如果只指定 `CHARACTER SET`，就使用 `CHARACTER SET` 默认的 `COLLATE`
- 如果 `CHARACTER SET` 和 `COLLATE` 都没有指定，就使用数据库默认的 `CHARACTER SET` 和 `COLLATE`

# 第 28 章 安全管理

## 访问控制

在开发环境和测试环境可以使用 root 用户，但是在生产环境绝对不能使用 root 用户

## 管理用户

## 创建用户

```sql
CREATE USER 用户名 IDENTIFIED BY '密码';
```

## 重命名用户

```sql
RENAME USER 旧用户名 TO 新用户名;
```

## 删除用户和关联的权限

```sql
DROP USER 用户名;
```

## 查看分配的权限

```sql
SHOW GRANTS FOR 用户名;
```

一个刚创建好的用户只能连接 MySQL，但是没有访问权限，因此看不到数据，也不能做任何操作

```
mysql> SHOW GRANTS FOR ben;
+---------------------------------+
| Grants for ben@%                |
+---------------------------------+
| GRANT USAGE ON *.* TO 'ben'@'%' |
+---------------------------------+
1 row in set (0.00 sec)
```

`USAGE` 关键字表示没有权限，后面的 `*.*` 是按照 `database.table` 格式写的，`*` 是通配符。`USAGE ON *.*` 表示对于任意的数据库和表都没有权限

`'ben'@'%'` 是按照 `user@host` 格式写的。如果 `host` 不指定就默认是 `%`，表示任意的 host

## 设置访问权限

要设置权限，至少要给出三个信息：

1. 要授予的权限
2. 被授予访问权限的数据库或表
3. 用户名

比如，给名叫 ben 的用户查询 crashcourse 数据的权限

```sql
GRANT SELECT ON crashcourse.* TO ben;
```

## 撤销权限

写法和设置访问权限一样，只需要把 `GRANT` 换成 `REVOKE`，`TO` 换成 `FROM` 即可

```sql
REVOKE SELECT ON crashcourse.* FROM ben;
```

## 更改密码

```sql
SET PASSWORD FOR 用户名 = Password('新密码');
```