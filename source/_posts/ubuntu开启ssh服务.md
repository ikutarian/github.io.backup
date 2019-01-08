---
title: ubuntu开启ssh服务
date: 2018-11-06 11:05:38
tags:
  - Ubuntu
  - ssh
categories:
  - 安装与配置
---

## 环境

- Ubuntu 16.04 64位
- xShell
- root权限

<!-- more -->

## 安装 openssh-server

首先更新一下软件源

```
$ apt-get update
```

安装 openssh-server

```
$ apt-get install openssh-server
```

## 查看查看ssh服务是否启动

```
$ ps -aux | grep ssh
```

输出

```
root       2783  0.0  0.3  92828  6708 ?        Ss   11:04   0:00 sshd: root@pts/0
```

能看到 sshd 说明服务启动了

## 查看本机的 ip

```
$ ifconfig
```

输出

```
ens32     Link encap:Ethernet  HWaddr 00:0c:29:e8:df:2c  
          inet addr:192.168.206.128  Bcast:192.168.206.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fee8:df2c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:925 errors:0 dropped:0 overruns:0 frame:0
          TX packets:481 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:996880 (996.8 KB)  TX bytes:50901 (50.9 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:176 errors:0 dropped:0 overruns:0 frame:0
          TX packets:176 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:13296 (13.2 KB)  TX bytes:13296 (13.2 KB)
```

`192.168.206.128` 就是本机的 ip 地址

## 连接 ssh

打开 xShell，配置 ip，输入账号密码即可连接

## Xshell连接Ubuntu时提示SSH服务器拒绝了密码

在使用 xShell 连接 root 用户是，报了“Xshell连接Ubuntu时提示SSH服务器拒绝了密码”这个错误。这是因为 ssh 默认不让 root 用户连接，需要改一下配置

首先备份 ssh 的配置

```
$ cp /etc/ssh/sshd_config /etc/ssh/sshd_config_backup
```

打开配置文件

```
$ vi /etc/ssh/sshd_config
```

找到：

```
# Authentication:
LoginGraceTime 120
PermitRootLogin prohibit-password
StrictModes yes
```

把 `PermitRootLogin` 的值修改成 `yes`，就是这样

```
# Authentication:
LoginGraceTime 120
PermitRootLogin yes
StrictModes yes
```

重启 ssh 服务

```
$ service ssh restart
```

现在可以通过 ssh 登陆 root 用户了