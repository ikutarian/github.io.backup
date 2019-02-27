---
title: 如何调试Dockerfile
date: 2019-02-27 16:23:24
tags:
  - Dockerfile
  - 调试
categories:
  - Docker
---

使用 Dockerfile 构建镜像时，如果出错了怎么办？怎么调试？

<!-- more -->

以构建 Nginx 镜像为例子来说明吧。这是一个 Dockerfile

```dockerfile
# 以 ikutarian/new_ubuntu 为基础镜像
FROM ikutarian/new_ubuntu

# 安装 Nginx
RUN apt-get install -y ngin

# 开放 80 端口
EXPOSE 80

# 容器启动时，执行的命令，这里是在前台启动 Nginx
CMD ["nginx", "-g", "daemon off"]
```

输入以下命令进行构建

```
docker build -t ikutarian/static_web .
```

控制台输出

```
root@ikutarian:~/docker/static_web# docker build -t ikutarian/static_web .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ikutarian/new_ubuntu
 ---> dc61113dcba0
Step 2/4 : RUN apt-get install -y ngin
 ---> Running in 85ae7d8b7fc0
Reading package lists...
Building dependency tree...
Reading state information...
E: Unable to locate package ngin
The command '/bin/sh -c apt-get install -y ngin' returned a non-zero code: 100
```

可以看到到了 `Step 2/4 : RUN apt-get install -y ngin` 这一步时出问题了，提示 `E: Unable to locate package ngin`，找不到 ngin，原来是 Nginx 的名字写错了，改一下 Dockerfile 文件

```dockerfile
# 以 ikutarian/new_ubuntu 为基础镜像
FROM ikutarian/new_ubuntu

# 安装 Nginx
RUN apt-get install -y nginx

# 开放 80 端口
EXPOSE 80

# 容器启动时，执行的命令，这里是在前台启动 Nginx
CMD ["nginx", "-g", "daemon off"]
```

然后重新构建

```
docker build -t ikutarian/static_web .
```

现在控制台输出

```
root@ikutarian:~/docker/static_web# docker build -t ikutarian/static_web .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ikutarian/new_ubuntu
 ---> dc61113dcba0
Step 2/4 : RUN apt-get install -y nginx
 ---> Running in ce4600296fa3
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:

省略....

Step 3/4 : EXPOSE 80
 ---> Running in a4680d0f7f3a
Removing intermediate container a4680d0f7f3a
 ---> 4bd0d0dc619f
Step 4/4 : CMD ["nginx", "-g", "daemon off"]
 ---> Running in aa753751d601
Removing intermediate container aa753751d601
 ---> 4c4afa69e432
Successfully built 4c4afa69e432
Successfully tagged ikutarian/static_web:latest
```

可以看到镜像创建成功了，输入 `docker images` 可以看到刚刚创建好的镜像

```
root@ikutarian:~/docker/static_web# docker images
REPOSITORY             TAG                 IMAGE ID            CREATED              SIZE
ikutarian/static_web   latest              4c4afa69e432        About a minute ago   199MB
```

启动容器

```
docker run --rm -d -p 80:80 ikutarian/static_web
```

然后访问宿主机的 IP 应该就能看到 Nginx 的页面了。但是却没有看到页面，而且执行 `docker ps` 也没有看到有容器在运行。应该是哪一个步骤出问题了，从 Dockerfile 文件和镜像构建时输出的信息来看，应该是在最后一步 `CMD ["nginx", "-g", "daemon off"]` 出了问题

由于镜像是分层的，Dockerfile 中的一行指令会生成一层，可以在 `CMD ["nginx", "-g", "daemon off"]` 的前一行指令输出如下

```
Step 3/4 : EXPOSE 80
 ---> Running in a4680d0f7f3a
Removing intermediate container a4680d0f7f3a
 ---> 4bd0d0dc619f
```

执行 `EXPOSE 80` 时，输出的 ID 是 `4bd0d0dc619f`，于是可以执行如下命令进入容器内部看一下

```
docker run -it -p 80:80 --rm 4bd0d0dc619f
```

然后执行一下 `CMD` 要执行的命令

```
root@9942a1275763:/# nginx -g daemon off
nginx: invalid option: "off"
```

可以看到提示，Nginx 提示命令不合法，所以就是这一行的 Nginx 启动命令错了。查了一下文档，正确的应该是

```
nginx -g "daemon off;"
```

现在知道问题在那里了，改一下 Dockerfile 文件

```dockerfile
# 以 ikutarian/new_ubuntu 为基础镜像
FROM ikutarian/new_ubuntu

# 安装 Nginx
RUN apt-get install -y nginx

# 开放 80 端口
EXPOSE 80

# 容器启动时，执行的命令，这里是在前台启动 Nginx
CMD ["nginx", "-g", "daemon off;"]
```

重新构建并运行容器

```
docker build -t ikutarian/static_web .
docker run --rm -d -p 80:80 ikutarian/static_web
```

## 总结

由于镜像是分层的，Dockerfile 中的一行指令会生成一层。所以可以在出错的指令前一层进入到容器内部调试