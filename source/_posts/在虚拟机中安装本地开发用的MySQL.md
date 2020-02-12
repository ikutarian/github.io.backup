---
title: 在虚拟机中安装本地开发用的MySQL
date: 2020-01-29 16:00:48
tags:  
  - MySQL
  - CentOS
  - 虚拟机
  - VMWare
  - 防火墙
  - SELinux
categories:
  - MySQL
---


本地开发与学习时，需要在虚拟机里的 CentOS 安装一个 MySQL 8。本文记录了当时的过程

<!-- more -->

# 安装 CentOS 7

## 下载 CenOS 镜像

本文使用的是阿里云镜像，使用以下地址下载

```
https://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-Everything-1908.iso
```

## 创建虚拟机

因为我的笔记本有 16G 内存，所以虚拟机的内存选择 2G

{% asset_img Snipaste_2020-01-29_16-08-46.png %}

CPU 1 颗，2 个核心，并且开启虚拟化

{% asset_img Snipaste_2020-01-29_16-09-11.png %}

网络适配器选择“桥接模式”，并且复制物理网络

{% asset_img Snipaste_2020-01-29_16-10-41.png %}

其他的无用硬件，比如声卡、打印机全部去掉

{% asset_img Snipaste_2020-01-29_16-11-21.png %}

## 安装过程

选择中文界面

{% asset_img Snipaste_2020-01-29_16-13-44.png %}

使用自动分区

{% asset_img Snipaste_2020-01-29_16-15-41.png %}

默认情况下 CentOS 是不会连接到网络的

{% asset_img Snipaste_2020-01-29_16-17-31.png %}

所以要手动打开

{% asset_img Snipaste_2020-01-29_16-16-44.png %}

# SSH 连接上 CentOS 7

输入以下命令获取 IP 地址

```
ip addr
```

然后再用 mobaxterm 或者 xShell 进行 SSH 连接即可

# 安装 MySQL

## 关闭SELinux

SELINUX 是 Linux 2.6 以上版本捆绑的安全模块。由于 SELinux 配置复杂，容易跟其他程序冲突，所以建议关闭

打开 SELinux 的配置文件

```
vi /etc/selinux/config
```

将 SELinux 设置为禁用

```
SELINUX=disabled
```

重启系统

```
reboot
```

这样 SELinux 就关闭了

## 在线安装 MySQL

### 下载 rpm 文件

这个文件记录了 MySQL 依赖的各种程序包

```
yum localinstall https://repo.mysql.com//mysql80-community-release-el7-1.noarch.rpm
```

### 安装 MySQL 数据库

```
yum install mysql-community-server -y
```

# 启动 MySQL

```
service mysqld start
```

# 查看 root 账户的临时密码

```
grep 'temporary password' /var/log/mysqld.log
```

# 重置 root 密码

首先登陆到 MySQL

```
mysql -u root -p
```

然后输入临时密码。接着更改密码

```
alter user user() identified by 'Abcd_12356';
```

# 允许 root 远程登陆 MySQL

## 允许远程使用 root 用户

```
USE mysql;
UPDATE user SET host = '%' WHERE user = 'root';
FLUSH PRIVILEGES;
```

## 修改 /etc/my.cnf 文件

```
character_set_server = utf8
bind-address = 0.0.0.0
```

## 重启 MySQL

```
service mysqld restart
```

# 开启系统防火墙的 3306 端口

开启 3306 端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
```

重启防火墙

```
firewall-cmd --reload
```

# 总结

1. CentOS 默认是不连接网络的，要手动开启
2. 善于使用 VMWare 虚拟机的快照功能，出错了可以快速恢复