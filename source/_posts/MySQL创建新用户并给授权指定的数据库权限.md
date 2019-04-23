---
title: MySQL创建新用户并给授权指定的数据库权限
date: 2019-04-04 17:09:22
tags:
  - MySQL
  - 权限
  - 安全
  - 授权
categories:
  - 安装与配置
---

一直依赖都是使用 `root` 用户操作数据库，今天被人批评了，这样很不好。应该新建一个用户，并指定该用户只能操作哪些数据库、表的权限，防止对其他数据进行非法操作。于是学习一下 MySQL 的用户、权限管理

<!-- more -->

# 前提

首先登陆到 root 用户

# 创建用户

比如我要创建一个名为 `ikutarian`，密码是 `123@456` 的用户

```sql
CREATE USER 'ikutarian'@'%' IDENTIFIED BY '123@456';
```

`'ikutarian'@'%'` 中的 `'ikutarian'` 就是用户名，`'%'` 表示用户的 IP 范围，一共有三种：

- `%`：任何 IP 都能登陆到这个用户
- `localhost`：只有本机才能登陆到这个用户
- `192.168.169.100`：指定的 IP 才能登陆到这个用户

# 给用户授权

授权就是只允许用户对某一个数据库做哪些操作。比如允许用户 `ikutarian` 对数据库 `study` 做所有的操作

```sql
GRANT ALL PRIVILEGES ON `mept`.* TO 'ikutarian'@'%';
```

现在退出 `root` 用户，使用 `ikutarian` 登陆到 MySQL，查看数据库

```sql
SHOW DATABASES;
```

只能看到这 3 个数据库

```
information_schema
study
test
```

# 可能遇到的问题

管理用户时，可能遇到不生效的问题，可以使用以下命令

```sql
FLUSH PRIVILEGES;
```

# 删除用户

使用 root 账号登陆

```sql
DROP USER ikutarian
```

# 修改密码

```sql
SET PASSWORD FOR ikutarian = Password('123456');
```

# 查看用户列表

使用 root 账号登陆

```sql
SELECT * FROM user;
```
