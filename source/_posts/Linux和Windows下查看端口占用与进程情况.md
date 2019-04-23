---
title: Linux和Windows下查看端口占用与进程情况
date: 2019-04-11 11:23:30
tags:
  - Linux
  - Windows
  - 端口
  - 端口占用
  - 进程
  - 命令行
categories:
  - 运维
---

开发与运维时，经常会遇到端口被占用的情况，有时候还需要查找端口是被谁占用了，有时候还需要查看进程是否启动了。这些情况都可以用命令行实现

<!-- more -->

# Linux 下

## 端口占用

使用 `netstat -nlp | grep [端口|程序名]` 命令，比如查看 80 端口的占用情况，输入以下命令

```
[root@asdf ~]# netstat -nlp | grep 80
tcp        0      0 0.0.0.0:80                0.0.0.0:*                   LISTEN      23534/nginx 
```

可以看到 80 端口被 Nginx 占用了，PID 是 `23534`，协议是 `tcp`，目前处于监听状态

说明一下几个参数的含义：

- `-n` 拒绝显示别名，能显示数字的全部转化为数字
- `-l` 仅列出在 Listening (监听)的服务状态
- `-p` 显示建立相关链接的程序名

可以用 `netstat -你老婆 | grep [端口|程序名]` 这个方法来记

## 进程情况

使用 `pf -ef | grep 进程名` 或者 `ps aux | grep 进程名` 查看相关信息，比如我要查看 Nginx 的进程信息

```
[root@asdf ~]# ps -ef | grep nginx
root      2160  2008  0 14:21 pts/0    00:00:00 grep nginx
root     23534     1  0 Mar27 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
nobody   31647 23534  0 10:05 ?        00:00:00 nginx: worker process 
```

# Windows 下

## 端口占用

输入 `netstat -ano` 可以看到所有端口的占用情况

```
C:\Users\test>netstat -ano

活动连接

  协议  本地地址          外部地址        状态           PID
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       1096
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       7152
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
```

假如 9527 端口被占用了，可以输入 `netstat -ano | findstr "9527"`

```
C:\Users\test>netstat -ano | findstr "9527"
  TCP    0.0.0.0:9527           0.0.0.0:0              LISTENING       8300
  TCP    127.0.0.1:61562        127.0.0.1:9527         TIME_WAIT       0
  TCP    127.0.0.1:61780        127.0.0.1:9527         TIME_WAIT       0
  TCP    [::]:9527              [::]:0                 LISTENING       8300
```

可以看到 9527 端口被 PID 为 8300 的进程占用了

## 进程情况

用 `tasklist` 查看进程列表

```
C:\Users\Think>tasklist

映像名称                       PID 会话名              会话#       内存使用
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          8 K
System                           4 Services                   0        160 K
```

如果想看 PID 为 8300 的进程的具体信息，可使用 `tasklist | findstr "8300"`

```
C:\Users\Think>tasklist | findstr "8300"
ShadowsocksR.exe              8300 Console                    2     23,756 K
```

可以看到是名为 ShadowsocksR.exe 应用启动的进程

# 总结

Linxu 下：

- `netstat -nlp` 结合 `grep` 使用
- `ps -ef` 或 `ps aux` 结合 `grep` 使用

Windows 下：

- `netstat -ano` 结合 `findstr` 使用
- `tasklist` 结合 `findstr` 使用