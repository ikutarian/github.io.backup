---
title: 什么时候用 INNER JOIN，什么时候用 LEFT JOIN
date: 2018-09-17 10:33:43
tags:
  - SQL
categories:
  - 数据库
---

首先准备两张表：

* 商品表：存放商品信息
* 图片表：存放图片

然后是建表语句。

商品表，商品的图片是关联到图片表的id

```sql
CREATE TABLE product (
    id int,
    name varchar(100) COMMENT '名称',
    img_id int COMMENT '关联的图片id'
);
```

图片表

```sql
CREATE TABLE image (
    id int,
    url varchar(500) COMMENT '图片的url'
);
```

<!-- more -->

图片表有4条数据

```sql
INSERT INTO image (id, url) VALUES 
(1, 'f7c27424b538281fee5f00db5f83c2de.jpg'),
(2, '522748524ad010358705b6852b81be4c.png'),
(3, '97adf9fdd4c3e8423f37a208beb47a7a.gif'),
(4, '7b1847d909f17365bec38b04b9da6e57.webp');
```

商品表里有4个商品，其中两个商品没有图片

```sql
INSERT INTO product (id, name, img_id) VALUES 
(1, '杯子', 1),
(2, '电脑', null),
(3, '大米饭', 3),
(4, 'PS4', null);
```

## 查询所有的商品的所有信息

列出所有的商品的所有信息，包括图片的 url。因为商品表的 `img_id` 关联到图片的 `id`，我首先想到的是这样的 sql 语句

```sql
SELECT
    product.id,
    product.name,
    image.url
FROM
    product,
    image
WHERE
    product.img_id = image.id
```

查询结果

|id|name|url|
|:--|:--|:--|
|1|杯子|f7c27424b538281fee5f00db5f83c2de.jpg|
|3|大米饭|97adf9fdd4c3e8423f37a208beb47a7a.gif|

这样的结果看起不对，没有列出所有的商品。因为商品的记录有4条，这里只列出了2条。另外两个没有图片的商品哪里去了？

## 正确的做法

正确的做法是列出所有的商品，如果图片不存在就显示为 `NULL`。这时候就可以使用左连接——`LEFT JOIN`

```sql
SELECT
    product.id,
    product.name,
    image.url
FROM
    product
LEFT JOIN image ON product.img_id = image.id
```

查询结果

|id|name|url|
|:--|:--|:--|	
|1|杯子|f7c27424b538281fee5f00db5f83c2de.jpg|
|3|大米饭|97adf9fdd4c3e8423f37a208beb47a7a.gif|
|2|电脑|NULL|
|4|PS4|NULL|

## 我的理解

两张表联合起来查询，如果左表要显示全部数据，同时右表有些字段没有值，那就用左连接，因为是以左表为主。相反情况，就用右连接。如果要求两张表都对上才显示数据，那就用内连接

上面我的错误代码

```sql
SELECT
    product.id,
    product.name,
    image.url
FROM
    product,
    image
WHERE
    product.img_id = image.id
```

其实就是内连接，要两张表对上才有数据显示

## INNER JOIN、LEFT JOIN 和 RIGHT JOIN 的区别

**(INNER) JOIN**：Returns records that have matching values in both tables
**LEFT (OUTER) JOIN**：Return all records from the left table, and the matched records from the right table
**RIGHT (OUTER) JOIN**：Return all records from the right table, and the matched records from the left table

{% asset_img 3617116-adf59d09aca742ef.png %}
