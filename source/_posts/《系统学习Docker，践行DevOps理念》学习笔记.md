---
title: 《系统学习Docker，践行DevOps理念》学习笔记
date: 2019-03-06 09:30:52
tags:
  - Docker
  - Dockerfile
categories:
  - Docker
---

本篇是学习《系统学习Docker，践行DevOps理念》的笔记

<!-- more -->

# DockerPlayground

如果电脑上没有安装 Docker 环境的话，可以在浏览器中使用 [DockerPlayground](https://labs.play-with-docker.com)。它提供了一个 Docker 环境供我们使用

# 删除已停止的容器

```
docker rm $(docker ps -f "status=exited" -q)
```

# Dockerfile 语法与最佳实践

## FROM

> Dockerfile 的第一行指令，用于选择 base image

示例：

```dockerfile
FROM scratch  # 使用空白镜像
FROM centos  # 使用最新版的 centos 镜像
FROM ubuntu:14.04  # 使用版本号为 14.04 的 ubuntu 镜像
```

最佳实践：

> 为了安全，尽量使用官方的 image 作为 base image

## LABEL

> 定义 image 的 metadata

示例：

```dockerfile
LABEL maintainer="ikutarian <ikutarian@ikutarian.com>"  # 维护者信息
LABEL version="1.0"  # 版本
LABEL description="This is description"  # 描述
```

最佳实践：

> metadata 不可少，要告诉别人这个镜像是做什么的

## RUN

> 执行命令并创建新的 image layer

示例：

```dockerfile
RUN yum update && yum install -y vim \
    python-dev  # 反斜线换行
RUN apt-get update && apt-get install -y perl \
    pwgen --no-install-recommends && rm -rf \
    /var/lib/apt/lists/*  # 注意清理apt-get 的 cache
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

最佳实践：

1. 命令尽量写在一行，可以用 `&&` 和 `\` 进行分割
2. 安装软件时，`apt-get update`、`apt-get install` 要写在一起，比如

```dockerfile
RUN apt-get update && apt-get install -y vim
```

3. 软件装完之后还要清理 apt-get 的缓存

```dockerfile
RUN apt-get update \
    && apt-get install -y vim \
    && apt-get clean
```

## WORKDIR

> 设定当前工作目录

```dockerfile
WORKDIR /test  # 如果没有 test 目录会自动创建
WORKDIR demo
RUN pwd  # 输出结果是 /test/demo
```

最佳实践：

> 1. 用 WORKDIR，不要用 `RUN cd`
> 2. 尽量使用绝对目录，这样更清晰不容易出错

## ADD 和 COPY

> 添加文件

最佳实践：

1. 大部分情况，COPY 优先于 ADD 使用，因为 COPY 语义更加直接
2. ADD 除了 COPY 还有额外功能（解压）
3. 添加远程文件/目录应该使用网 `RUN wget` 或者 `RUN curl` 命令来获取。因为使用 `wget` 或者 `curl` 更加方便清理存储的中间文件和临时文件，同时这样也能使用更少的 layer 来完成同样的事情

## ENV

> 设置环境变量

```dockerfile
ENV MYSQL_VERSION 5.6  # 设置常量
RUN apt-get update && apt-get install -y mysql-server = "${MYSQL_VERSION}" \
    && rm -rf /var/lib/apt/lists/*
```

最佳实践：

尽量使用 ENV 来增加可维护性

## CMD 与 ENTRYPOINT

看这篇文章《{% post_link Docker中RUN、CMD和ENTRYPOINT的区别 %}》