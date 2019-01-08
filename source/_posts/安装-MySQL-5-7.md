---
title: 安装 MySQL 5.7
date: 2018-09-14 17:22:45
tags:
  - MySQL
categories:
  - 安装与配置
---

自MySQL版本升级到5.7以后，其安装及配置过程和原来版本发生了很大的变化，下面详细介绍5.7版本MySQL的下载、安装及配置过程。

## 版本

目前针对不同用户，MySQL提供了2个不同的版本：

* MySQL Community Server：社区版，该版本完全免费，但是官方不提供技术支持。
* MySQL Enterprise Server：企业版，它能够高性价比的为企业提供数据仓库应用，支持ACID事物处理，提供完整的提交、回滚、崩溃恢复和行级锁定功能。但是该版本需付费使用，官方提供电话及文档等技术支持。

本文使用的是**社区版**。

<!-- more -->

## 下载

下载地址：https://dev.mysql.com/downloads/

选择 MySQL Community Server

{% asset_img 3617116-c5ef6fa88dc9377c.png %}

因为我的系统是 windows 7 64 位，所以选择系统对应的版本

{% asset_img 3617116-800a4494c8c88856.png %}

下载好得到了 mysql-5.7.17-winx64.zip 压缩包

## 解压

将压缩包解压到 D:\mysql-5.7.17-winx64

## 配置环境变量

 右键*计算机 -> 属性 -> 高级系统设置 -> 环境变量*

在系统变量里添加 `MYSQL_HOME` 环境变量，变量值为MySQL的根目录（也就是压缩包解压的位置：D:\mysql-5.7.17-winx64）

{% asset_img 3617116-05af4e4d1495751a.png %}

找到path，选择编辑，在原有值末尾添加 `;%MYSQL_HOME%\bin`

{% asset_img 3617116-d262c87f5634eb48.PNG %}

## 添加配置文件

在MySQL的安装目录（例如我的是D:\mysql-5.7.17-winx64）下，建立新文本文件txt，并将其命名为my.ini（注意扩展名也要修改）

在其中加入如下内容

```ini
[mysqld]
basedir=D:\mysql-5.7.17-winx64
datadir=D:\mysql-5.7.17-winx64\data
port=3306

character-set-server=utf8
collation-server=utf8_general_ci
```

## 初始化

以管理员身份打开 CMD 执行以下命令（注意必须以管理员身份打开，否则报错）

```
mysqld --initialize --user=mysql --console
```

在控制台消息尾部会出现随机生成的初始密码，记下来（因为有特殊字符，很容易记错，最好把整个消息保存在记事本里）

下面是输出结果

```
D:\mysql-5.7.17-winx64>mysqld --initialize --user=mysql --console
2017-04-07T01:52:43.345565Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is
 deprecated. Please use --explicit_defaults_for_timestamp server option (see doc
umentation for more details).
2017-04-07T01:52:45.657697Z 0 [Warning] InnoDB: New log files created, LSN=45790

2017-04-07T01:52:45.966715Z 0 [Warning] InnoDB: Creating foreign key constraint
system tables.
2017-04-07T01:52:45.999717Z 0 [Warning] No existing UUID has been found, so we a
ssume that this is the first time that this server has been started. Generating
a new UUID: e5751f68-1b34-11e7-9447-00e066f46b1d.
2017-04-07T01:52:46.029718Z 0 [Warning] Gtid table is not ready to be used. Tabl
e 'mysql.gtid_executed' cannot be opened.
2017-04-07T01:52:46.192728Z 1 [Note] A temporary password is generated for root@
localhost: wdu*Ye<vW25)

D:\mysql-5.7.17-winx64>
```

可以看到最后一行

```
localhost: wdu*Ye<vW25)
```

其中 `wdu*Ye<vW25)` 就是随机密码

## 将MySQL添加到系统服务

以管理员自身份打开CMD执行以下命令（注意必须以管理员身份打开，否则报错）

```
mysqld --install MySQL
```

出现

```
D:\mysql-5.7.17-winx64>mysqld --install MySQL
Service successfully installed.
```

说明服务安装成功。

安装成功之后，就是启动 MySQL 服务，输入以下命令

```
net start MySQL
```

出现

```
D:\mysql-5.7.17-winx64>net start MySQL
MySQL 服务正在启动 .
MySQL 服务已经启动成功。
```

说明启动成功

## 启动MySQL并修改密码

在CMD控制台里执行命令

```
mysql -u root -p
```
回车执行后，输入刚才记录的随机密码（`wdu*Ye<vW25)`）

执行成功后，控制台显示 mysql>，则表示进入mysql
输入命令

```
set password for root@localhost = password('123'); 
```
（注意分号）。此时root用户的密码修改为123

## 下一次如何进入 MySQL

下一次打开电脑的时候，先启动 MySQL 服务

```
net start MySQL
```

然后再登录到 MySQL

```
mysql -u root -p
```

输入密码 123

## 如下密码忘记了怎么办

首先停止服务

```
net stop MySQL
```

修改 my.ini 文件，加一条

```
[mysqld]
basedir=D:\mysql-5.7.17-winx64
datadir=D:\mysql-5.7.17-winx64\data
port=3306
skip-grant-tables
```

再重新启动服务

```
net start MySQL
```

登录到 MySQL

```
mysql -u root -p
```

不需要输入密码，直接输入回车，进入 MySQL 的控制台

接着输入如下 SQL 语句重置密码

```
USE mysq;
UPDATE user SET authentication_string=password('新密码') WHERE user='root';
```

现在密码已经重置好了，那么就需要将 my.ini 改回来，把 skip-grant-tables 注释掉就好了

```
[mysqld]
basedir=D:\mysql-5.7.17-winx64
datadir=D:\mysql-5.7.17-winx64\data
port=3306
#skip-grant-tables
```

最后就是重新启动 MySQL 服务

```
net stop MySQL
net start MySQL
```

输入新密码登录到 MySQL

```
mysql -u root -p
```