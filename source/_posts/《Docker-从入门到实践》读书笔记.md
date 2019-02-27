---
title: 《Docker 从入门到实践》读书笔记
date: 2018-11-10 14:14:57
tags:
  - Docker
categories:
  - 读书笔记
---

读书计划开始了，这是我要阅读的第三本书

<!-- more -->

# 关于这本书

可以在《[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/content/)》看到原文。也可以直接在[这里](/uploads/docker_practice.pdf)下载

# Docker 简介

## 什么是 Docker

- Go 语言开发
- 操作系统层面的虚拟化技术

## 为什么要用 Docker

1. 不需要进行硬件虚拟化和运行完整的操作系统
2. 启动很快，秒级
3. 一致的运行环境
4. DevOps
5. 迁移很方便
6. 维护和扩展很方便

# 核心概念

- 镜像（Image）
- 容器（Container）
- 仓库（Repository）

镜像（Image）可以理解成一个里面有一个操作系统和一些软件的安装包

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样。我可以通过安装镜像，得到一个能运行的软件

仓库（Repository）就是存放镜像（Image）的仓库，仓库有公共仓库和私有仓库两种

**一个容易混淆的概念**

经常有人把仓库（Repository）和仓库注册服务器（Registry）混为一谈。两者是不同的东西。仓库（Repository）放镜像；仓库注册服务器（Registry）放仓库。比如 Ubuntu 镜像就放在 Ubuntu 的仓库里，按照不同的 TAG，比如 14.04、16.04、18.04 等来区分不同版本的 Ubuntu。仓库注册服务器（Registry）有很多的仓库，比如 Ubuntu 的仓库、Nginx 的仓库、Tomcat 的仓库、MySQL 的仓库等等

# 安装

看这篇文章《{% post_link Ubuntu安装Docker-CE %}》

# 帮助文档

docker 命令的文档，有两种方式可以查看。

1. 按照 `docker xxx --help` 这样的格式即可查看简略文档，比如 `docker pull --help`
2. 可以通过 `man docker xxx` 查看详细文档。比如要查看 `docker pull` 命令的文档，可以输入 `man docker pull`

目前 Docker 支持这些命令

```
Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

# Docker 镜像的操作

常用的操作有：

- 拉取镜像
- 列出本地的镜像
- 查看镜像的详细信息
- 删除本地镜像
- 镜像导入导出

## 拉取镜像

镜像有名字，格式为：`NAME:TAG`，如果 `TAG` 不指定的话，默认是 `latest`

```
docker pull NAME[:TAG]
```

比如

```
docker pull ubuntu:16.04
```

会拉取 ubuntu 仓库中 TAG 为 16.04 的镜像

```
docker pull ubuntu
```

没有指定 TAG，默认为 `latest`，因此会拉取 ubuntu 仓库中最新的镜像

## 列出本地的镜像

```
docker images
```

或者

```
docker image ls
```

可以列出已经从仓库拉取保存到本地的镜像

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
gitlab/gitlab-ce    latest              446c42fd5226        10 days ago         1.56GB
ubuntu              16.04               4a689991aa24        2 weeks ago         116MB
ubuntu              latest              ea4c82dcd15a        2 weeks ago         85.8MB
nginx               latest              dbfc48660aeb        2 weeks ago         109MB
jenkins/jenkins     lts                 9cff19ad8c8b        3 weeks ago         730MB
registry            latest              2e2f252f3c88        7 weeks ago         33.3MB
hello-world         latest              4ab4c602aa5e        8 weeks ago         1.84kB

```

可以看到镜像的一些信息：

- REPOSITORY：镜像的仓库名称
- TAG：镜像的 TAG
- IMAGE ID：镜像的唯一标识
- CREATED：镜像的创建时间
- SIZE：镜像的大小

也可以指定只列出某一个仓库的镜像，比如只列出 ubuntu 仓库的镜像

```
docker image ls ubuntu
```

输出

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               4a689991aa24        2 weeks ago         116MB
ubuntu              latest              ea4c82dcd15a        2 weeks ago         85.8MB
```

有一个命令参数很有用：`-q`，只列出镜像的 `IMAGE_ID`。比如

列出所有镜像的 `IMAGE_ID`

```
docker image ls -q
```

列出 ubuntu 仓库中所有镜像的 `IMAGE_ID`

```
docker image ls ubuntu -q
```

## 查看镜像的详细信息

```
docker inspect NAME[:TAG] | IMAGE_ID
```

## 删除本地镜像

可以使用 `NAME[:TAG]`（仓库名：标签）或者 `IMAGE_ID`（镜像ID）来删除镜像

```
docker image rm NAME[:TAG] | IMAGE_ID
```

`docker image rm` 可以简写为 `docker rmi`

之前说过，可以用 `docker image ls` 带上参数 `-q` 可以列出镜像的 `IMAGE_ID`。如果要删除全部镜像，命令还可以这么写

```
docker image rm $(docker image ls -q)
```

## 镜像导入导出

导出

```
docker save ubuntu > ubuntu.tar
```

导入

```
docker load < ubuntu.tar
```

# 用 Dockerfile 定制自己的镜像

**这是重点！**

镜像的定制实际上就是定制每一层所添加的配置、文件，把每一层修改、安装、构建、操作的命令都写入一个名为 Dockerfile 的脚本，用这个脚本来构建、定制镜像

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，**每一条指令构建一层**。每一条指令的内容，就是描述该层应当如何构建。Dockerfile 的写法可以看 [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)，最佳实践可以看 [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## 一个例子

以自己定制一个 nginx 镜像为例。创建一个文件夹，在文件夹新建一个名为 `Dockerfile` 的文件（这一步很重要，每次定制镜像时，都建立一个独立的文件夹）

```
mkdir mynginx
cd mynginx
touch Dockerfile
```

Dockerfile 的内容为

```Dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

构建镜像（别忘了后面有一个 `.`）

```
docker build -t nginx:ikutarian .
```

会输出

```
Sending build context to Docker daemon  2.048kB
Step 1/2 : FROM nginx
latest: Pulling from library/nginx
f17d81b4b692: Pull complete 
d5c237920c39: Pull complete 
a381f92f36de: Pull complete 
Digest: sha256:b73f527d86e3461fd652f62cf47e7b375196063bbbd503e853af5be16597cb2e
Status: Downloaded newer image for nginx:latest
 ---> dbfc48660aeb
Step 2/2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 374b25943bd3
Removing intermediate container 374b25943bd3
 ---> a86dfb4b5ce9
Successfully built a86dfb4b5ce9
Successfully tagged nginx:ikutarian
```

这样一个仓库名为 `nginx`，标签为 `ikutarian` 的镜像就创建好了。输入 `docker images nginx` 就可以看到新增了一个 `nginx:ikutarian` 镜像

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               ikutarian           a86dfb4b5ce9        2 minutes ago       109MB
nginx               latest              dbfc48660aeb        3 weeks ago         109MB
```

# 容器操作

- 启动（**这是重点**）
  - 后台启动
  - 进入容器
- 查看容器列表
- 停止
- 重启
- 删除

## 启动

有两种方式：

1. 基于镜像新建一个容器并启动

```
docker run -i -t ubuntu bash
```

`-i` 表示开启 STDIN；`-t` 表示开启一个伪 tty 终端；指定一个名字为 `ubuntu` 的镜像来创建容器；`bash` 是启动容器之后要执行的命令，表示启动了一个 bash shell

2. 将已经停止的容器重新启动

输入

```
docker ps -a
```

输出

```
root@iZ94fg0bhwgrgtZ:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
05d194951f12        ubuntu              "bash"              25 seconds ago      Exited (0) 5 seconds ago                       nostalgic_thompson
```

找一个已经处于停止状态的容器。这里正好有一个 `STATUS` 是 `Exited (0) 5 seconds ago` 的容器，ID 是 `05d194951f12`，那么就可以

```
docker start 05d194951f12
```

把这个容器启动起来

### 后台启动

可以加上参数 `-d` 让容器在后台运行

```
docker run -it -d ubuntu bash
```

docker 会返回一个很长的容器 ID

```
root@iZ94fg0bhwgrgtZ:~# docker run -itd ubuntu bash
9fd01d0651556d6bbfa3e7606b56e96983825d20a519727557ee0f7805265bad
```

### 进入容器

进入容器有两种方法：

1. docker attach CONTAINER_ID
2. docker exec CONTAINER_ID

`docker attach IMAGE_ID` 如果输入 `exit` 退出虚拟终端会把容器给停掉。`docker exec IMAGE_ID` 则相反，输入 `exit` 不会把容器停掉。所以推荐使用 `docker exec IMAGE_ID`

可以看下面的例子

列出正在运行的容器

```
root@iZ94fg0bhwgrgtZ:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
9fd01d065155        ubuntu              "bash"              5 minutes ago       Up 5 minutes                            unruffled_yonath
```

进入一个正在运行的容器

```
root@iZ94fg0bhwgrgtZ:~# docker exec -it 9fd bash
```

在容器中输入 `exit` 退出终端

```
root@9fd01d065155:/# exit
exit
```

再次查看正在运行的容器，可以看见容器还是在正常运行，没有停止

```
root@iZ94fg0bhwgrgtZ:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
9fd01d065155        ubuntu              "bash"              6 minutes ago       Up 6 minutes                            unruffled_yonath
```

## 查看容器列表

查看正在运行的容器

```
docker ps
```

查看所有容器，不管是停止还是正在运行的

```
docker ps -a
```

会输出

```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
bc11b336f77e        ubuntu              "/bin/bash"         11 minutes ago      Exited (127) 7 seconds ago                       dazzling_northcutt
```

**注意到一个现象**

之前输入

```
docker run -i -t ubuntu bash
```

启动一个 ubuntu 镜像，并进入 bash shell。用户敲了一些命令，执行了一些任务之后。输入 `exit` 退出 ubuntu。这时候再输入

```
docker ps
```

却发现没有看到 ubuntu 出现在列表中。这是为什么呢？

> 这是因为想要保持 Docker 容器的活跃状态，需要其中运行前台的进程不能中断。中断了话容器就停止了

## 停止

```
docker stop CONTAINER_ID
```

## 重启

```
docker restart CONTAINER_ID
```

## 删除

```
docker rm CONTAINER_ID
```

可以利用命令参数 `-q`，一条命令删除所有容器

```
docker rm $(docker ps -aq)
```

# 数据管理

两种方式：

1. 数据卷（Volumes）
2. 挂载宿主机目录（Bind mounts）

# 使用网络

- 外部访问容器
- 容器互联

## 外部访问容器

“外部访问容器”就是要让外部的请求通过端口映射转发到容器内部。使用 `-p` 参数可以指定要映射的端口，格式有：

- 宿主机端口:容器端口
- 宿主机ip:宿主机端口:容器端口
- 宿主机ip::容器端口

### 宿主机端口:容器端口

宿主机端口 5000 映射到容器的 5000 端口

```
docker run -p 5000:5000 
```

### 宿主机ip:宿主机端口:容器端口

宿主机的 localhost:5000 映射到容器的 5000 端口

```
docker run -p 127.0.0.1:5000:5000
```

### 宿主机ip::容器端口

宿主机的 localhost 所有端口的请求都转发到容器的 5000 端口

```
docker run -p 127.0.0.1::5000
```

### 可以一次映射多个端口

```
docker run -p 5000:5000 -p 3000:80
```

## 容器互联

“容器互联”就是要让容器与容器之间互相通信。

# 总结

这本书的学习就到这里了，后面的知识和 Docker 没什么太大关系，现在学也没什么用。接下来就是实践，边做边学