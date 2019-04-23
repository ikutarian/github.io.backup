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

## Linux下的安装与启动

### 环境

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

## Windows下的安装与启动

### 环境

- 操作系统：Windows 10
- Redis版本：Redis-x64-3.2.100

### 安装步骤

打开 [Redis for Windows](https://github.com/MicrosoftArchive/redis/releases) 的下载地址，选择一个版本下载。下载完了之后，解压到某一个目录下，目录不要有中文和空格

### 配置文件

安装包目录下已经有一个名字是 `redis.windows.conf` 的配置文件，可以直接拿来用

### 启动Redis

```
redis-server  redis.windows.conf
```

如果成功启动可以看到这个界面

```
E:\Redis-x64-3.2.100>redis-server.exe redis.windows.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.100 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 8592
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[8592] 02 Apr 10:17:00.744 # Server started, Redis version 3.2.100
[8592] 02 Apr 10:17:00.748 * The server is now ready to accept connections on port 6379
```

### 启动Redis客户端

双击 `redis-cli.exe`, 如果不报错的话会显示

```
127.0.0.1:6379>
```

### 关闭Redis

直接按下 `CTRL + C` 即可停止 Redis 服务端