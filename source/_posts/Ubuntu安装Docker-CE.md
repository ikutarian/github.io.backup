---
title: Ubuntu安装Docker-CE
date: 2018-11-01 14:29:09
tags:
  - Docker
  - Ubuntu
categories:
  - 安装与配置
---

Docker 有两个版本: 

- Docker-CE (社区版)
- Docker-EE (企业版)

我将在 Ubuntu 16.04 系统下安装 Docker-CE。关于如何安装，官方有一份说明文档 [Get Docker CE for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。现在根据文档，按照我的方式来总结一下

<!-- more -->

## 系统要求

目前支持的 Ubuntu 版本有：

- Bionic 18.04 (LTS)
- Xenial 16.04 (LTS)
- Trusty 14.04 (LTS)

要查看自己的版本可以输入

```
$ lsb_release -a
```

我的版本是

```
LSB Version:	core-9.20160110ubuntu0.2-amd64:core-9.20160110ubuntu0.2-noarch:security-9.20160110ubuntu0.2-amd64:security-9.20160110ubuntu0.2-noarch
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.4 LTS
Release:	16.04
Codename:	xenial
```

如果 `lsb_release` 命令无法使用的话，可以使用

```
$ cat /etc/os-release
```

输出

```
NAME="Ubuntu"
VERSION="16.04.4 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.4 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
```

## 删除老版本的 Docker

老版本的 Docker 叫 `docker` 或者 `docker-engine`，如果有安装的话，要先删除

```
$ sudo apt-get remove docker docker-engine docker.io
```

## 安装方式

官方提供了 3 种方式

- Docker's repositories
- DEB package
- automated convenience scripts

推荐使用 `Docker's repositories` 方式

## 开始安装

### 1. 更新软件源

```
$ sudo apt-get update
```

### 2. 让 APT 支持 HTTPS

```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

### 3. 添加 Docker 的官方密钥

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

显示 OK，表示添加成功

### 4. 添加 Docker 的 **stable** repository

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

### 5. 再次更新软件源

```
$ sudo apt-get update
```

### 6. 安装 Docker-CE

现在终于可以安装 Docker-CE 了

```
$ sudo apt-get install docker-ce
```

如果要安装特定版本的 Docker-CE，可以这么做：

1. 首先查看 repository 里有哪些版本的 Docker-CE

```
$ apt-cache madison docker-ce
```

会列出来一些版本

```
docker-ce | 18.06.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
...
```

其中 `18.06.1~ce~3-0~ubuntu` 就是版本的名称

2. 安装

把 `<VERSION>` 换成版本名称就行

```
$ sudo apt-get install docker-ce=<VERSION>
```

### 7. 验证

```
$ sudo docker run hello-world
```

可以看到一大串 Docker 输出的信息就表示安装成功了

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
d1725b59e92d: Pull complete 
Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## 更新

把 `安装` 步骤再走一遍，选择要安装的版本即可

## 卸载

卸载 Docker-CE

```
$ sudo apt-get purge docker-ce
```

删除所有的 images、containers 和 volumes

```
$ sudo rm -rf /var/lib/docker
```

## 镜像加速

国内访问 Docker Hub 的话，速速比较慢，可以使用一些厂商搭建好的镜像进行加速。厂商有很多

- [Docker Hub 的中国镜像](https://registry.docker-cn.com)
- [阿里云](https://cr.console.aliyun.com/#/accelerator)
- [DaoCloud](https://www.daocloud.io/mirror#accelerator-doc)

打开 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在就新建一个），以 Docker Hub 中国镜像为例

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}
```

之后重新启动服务

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```