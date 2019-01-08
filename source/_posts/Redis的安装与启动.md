---
title: Redis的安装与启动
date: 2018-09-15 14:55:30
tags:
  - Redis
categories:
  - 安装与配置
---

## 什么是Redis

* Redis是远程的，C/S
* Redis是基于内存的
* Redis是非关系型数据库

## Redis的应用场景

* 缓存
* 队列
* 数据存储

## Redis安装

### 安装环境

1. 服务器环境：Ubuntu 64
2. Redis版本：4.0.8
3. 预装软件：gcc，tcl

<!-- more -->

### 安装步骤

```
$ wget http://download.redis.io/releases/redis-4.0.8.tar.gz
$ tar xzf redis-4.0.8.tar.gz
$ cd redis-4.0.8
$ make
$ make test
$ make install
```

如果在运行 `make test` 命令时，报错

```
cd src && make test
make[1]: Entering directory '/root/download/redis-3.0.7/src'
You need tcl 8.5 or newer in order to run the Redis test
Makefile:211: recipe for target 'test' failed
make[1]: *** [test] Error 1
make[1]: Leaving directory '/root/download/redis-3.0.7/src'
Makefile:6: recipe for target 'test' failed
make: *** [test] Error 2
```

可以看到报错信息中提示

```
You need tcl 8.5 or newer in order to run the Redis test
```

所以需要安装一下 `tcl`，输入 `sudo apt-get install tcl` 即可安装。我在运行 `sudo apt-get install tcl` 命令时，提示 package 找不到

```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package tcl
```

可以更新一下软件源，输入 `apt-get update`，然后再输入 `sudo apt-get install tcl` 即可安装 tcl

### 配置文件

首先复制一份redis的配置文件

```
cp redis-4.0.8/redis.conf ~/conf/redis.conf
```

然后修改配置文件

```
port 7200  # 修改端口
daemonize yes  # 后台启动
```

### 启动Redis

```
redis-server ~/conf/redis.conf
```

### 启动Redis客户端

```
redis-cli -h 127.0.0.1 -p 7200
```

可以通过 `ps aux|grep redis-server` 查看redis的进程情况

### 关闭Redis

```
redis-cli shutdown
```