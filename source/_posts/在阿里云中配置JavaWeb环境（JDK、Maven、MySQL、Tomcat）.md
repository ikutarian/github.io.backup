---
title: 在阿里云中配置-JavaWeb-环境（JDK、Maven、MySQL、Tomcat）
date: 2018-09-15 14:57:44
tags:
  - JDK
  - Maven
  - MySQL
  - Tomcat
  - 阿里云
categories:
  - 安装与配置
---

## 系统环境

* 阿里云
* ubuntu 16.04 64位

## 前期配置

首先分别创建一下文件下载的存放目录。目的是为了更好的管理

```
cd ~
mkdir -p download/jdk
mkdir -p download/maven
mkdir -p download/tomcat
```

然后再创建软件的安装目录

```
mkdir -p app/jdk
mkdir -p app/maven
mkdir -p app/tomcat
```

<!-- more -->

## JDK

首先百度一下 JDK

{% asset_img 3617116-d5907c0458d3ca79.png %}

选择这个链接进入 JDK 的下载网站，然后再获取到 JDK 的下载地址。

> 直接点击右键“复制链接地址”可能是无效的，所以可以先点击下载，然后在下载器中复制下载的地址

{% asset_img 3617116-56d5c32464acbca6.png %}

得到下载地址 
```
http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz?AuthParam=1530779979_b993d1478f9836e5828b9cea142a7b11
```

打开 ubuntu 的控制台，输入

```
wget http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz?AuthParam=1530779979_b993d1478f9836e5828b9cea142a7b11 -O jdk-8u171-linux-x64.tar.gz
```

即可下载 jdk 并以 jdk-8u171-linux-x64.tar.gz 为文件名保存到磁盘中

接着是解压

```
tar -xzvf jdk-8u171-linux-x64.tar.gz
```

然后把解压后得到的文件夹 jdk1.8.0_171 移动到 ~/app/jdk 下

```
mv jdk1.8.0_171/ ~/app/jdk/
```

编辑 /etc/profile，在最下方添加：

```
JAVA_HOME=/root/app/jdk/jdk1.8.0_171
export JAVA_HOME
export PATH=${PATH}:${JAVA_HOME}/bin
```

保存文件，并运行如下命令使环境变量生效：

```
source /etc/profile
```

检查 Java 是否成功安装

```
javac
```

## Maven

从[官网下载页](https://maven.apache.org/download.cgi)获取最新的下载链接

然后使用 wget 命令将其下载：

```
wget http://mirrors.shu.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
```

解压压缩包：

```
tar -xzvf apache-maven-3.5.4-bin.tar.gz
```

将文件夹移动至 ~/app/maven 目录：

```
mv apache-maven-3.5.4 ~/app/maven/
```

配置环境变量

编辑 /etc/profile，在最下方添加：

```
MAVEN_HOME=/root/app/maven/apache-maven-3.5.4
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

保存文件，并运行如下命令使环境变量生效：

```
source /etc/profile
```

检查 Maven 是否成功安装：

```
mvn -version
```

## Tomcat

由于 JDK 安装的是 8.0 版本，于是 Tomcat 也选择 8.0 版本。

同样的，在[下载页](https://tomcat.apache.org/download-80.cgi)获取 tomcat8 的下载地址，然后下载保存到磁盘中

```
wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.32/bin/apache-tomcat-8.5.32.tar.gz
```

解压压缩包：

```
tar -zxvf apache-tomcat-8.5.32.tar.gz
```

将文件夹移动至 ~/app/tomcat 目录：

```
mv apache-tomcat-8.5.32 ~/app/tomcat/
```

进入 ~/app/tomcat/

```
cd ~/app/tomcat/
```

启动 Tomcat

```
./startup.sh
```

关闭 Tomcat

```
./shutdown.sh
```

阿里云默认 8080 端口是不开放的，所以需要配置一下安全组规则。按照下图配置一下就好了。

{% asset_img 3617116-dd428a0b140c7a42.png %}

然后启动 Tomcat，在浏览器中输入 http://阿里云ip:8080 即可访问到 Tomcat 了

## MySQL

直接输入下面命令安装最新版的 MySQL

```
apt-get install mysql-server mysql-client
```

就这就会出现输入 root 密码的界面，输入密码即可

启动 MySQL

```
service mysql start
```

关闭 MySQL

```
service mysql stop
```

