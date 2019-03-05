---
title: Docker镜像构建实例
date: 2019-02-26 14:29:04
tags:
  - 镜像
  - 容器
  - 实例
  - Dockerfile
categories:
  - Docker
---

用实际例子来学习 Docker

<!-- more -->

# 安装好 Vim 的 Ubuntu 镜像

Ubuntu 的 base 镜像默认是没有安装 Vim 软件的，现在来制作一个装好 Vim 软件的镜像

新建一个文件夹，比如 `test`

```
mkdir test && cd test
```
 
新建一个名为 `Dockerfile` 的文件，写入如下内容
 
```
FROM ubuntu
RUN apt-get update && apt-get install -y vim
```

Dockerfile 中命令的含义是：

1. 以 ubuntu 为 base 镜像
2. 先执行 `apt-get update` 命令更新软件源
3. 然后再执行 `apt-get install -y vim` 命令安装 vim（`-y` 参数的意思是不询问直接安装软件）

编译 Dockerfile，把新镜像命名为 `ubuntu-vi`

```
docker build -t ubuntu-vi .
```

执行 `docker images` 就能看到一个名为 `ubuntu-vi` 的镜像了

# 第一个 Docker 化的 Java 应用

[JPress](http://www.jpress.io/) 是 Java 版的 WordPress。现在使用 Docker 来构建一个 JPress 的应用

过程如下：

1. 使用 Tomcat 的镜像，把 JPress 的 WAR 包打包进去
2. 再启动一个 MySQL 的 Docker 容器
3. JPress 应用连接 MySQL

## 配置阿里云安全组规则

阿里云默认只允许 22 端口连接，如果还需要其他端口的话，需要配置一下安全组规则。这里需要配置一下 Tomcat 的 8080 端口和 MySQL 3306 端口

### 8080 端口

在实例列表中，选择“更多 - 安全组配置”

{% asset_img Snipaste_2019-02-26_14-01-50.png %}

选择“配置规则”

{% asset_img Snipaste_2019-02-26_14-03-24.png %}

点击右上角的“添加安全组规则”

{% asset_img Snipaste_2019-02-26_14-06-40.png %}

按照如下图设置，允许外网访问 8080 端口

{% asset_img Snipaste_2019-02-26_14-09-21.png %}

对于“规则方向”，[文档](https://help.aliyun.com/document_detail/25471.html?spm=5176.2020520101.0.d0.7ce34df5gZH4NP) 里有讲：

- 出方向：是指 ECS 实例访问内网中其他 ECS 实例或者公网上的资源。（也就是主机到外网）
- 入方向：是指内网中的其他 ECS 实例或公网上的资源访问 ECS 实例。（即外网到主机）

### 3306 端口

按照下图设置

{% asset_img Snipaste_2019-02-26_14-20-20.png %}

需要注意的是，我只允许本机可以访问 MySQL，所以把“授权对象”填写成自己的外网 IP

## 构建 JPress 镜像

新建一个文件夹，然后下载 JPress 的 WAR 包，改名为 `jpress.war`

```
mkdir jpress
cd jpress
wget https://raw.githubusercontent.com/JpressProjects/jpress/alpha/wars/jpress-web-newest.war
mv jpress-web-newest.war jpress.war
```

创建一个 Dockerfile 文本文件，然后写入如下信息。tomcat 镜像的配置，可以看[官网](https://hub.docker.com/_/tomcat)

```dockerfile
# 使用 tomcat 作为基础镜像
FROM tomcat:8

# 作者信息
LABEL maintainer="ikutarian46@ikutarian46.com"

# 把 jpress.war 复制到 webapps 目录下
COPY jpress.war $CATALINA_HOME/webapps
```

构建镜像，镜像的名字指定为 `jpress:latest`

```
docker build -t jpress:latest .
```

输入 `docker image ls jpress` 就可以看到创建好的 jpress 镜像了

```
root@iZ94fg0bhwgrgtZ:~/docker# docker image ls jpress
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jpress              latest              adda9c53bd34        3 minutes ago       483MB
```

## 启动 JPress 镜像

```
docker run -d -p 8080:8080 jpress
```

`-d` 表示容器后台启动，`-p 8080:8080` 表示宿主机的 8080 端口访问容器的 8080 端口。TAG 默认是 `latest`，所以不写也行

使用 `netstat -na | grep <port>` 可以查看宿主机的端口监听状态。使用 `docker ps` 也可以看到目前启动的容器相关信息

## 启动 MySQL 容器

mysql 镜像的使用方法，可以看[官网](https://hub.docker.com/_/mysql)

```
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=jpress mysql:5.7
```

`-d` 后台启动。`-p 3306:3306` 宿主机的 3306 端口连接容器的 3306 端口。`-e MYSQL_ROOT_PASSWORD=123456` 指定环境变量 `MYSQL_ROOT_PASSWORD` 的值，这里是指定 ROOT 密码为 `123456`。`-e MYSQL_DATABASE=jpress` 指定环境变量 `MYSQL_DATABASE` 的值，这里是默认创建一个名为 jpress 的数据库

## 使用 JPress

根据本机的 IP 访问 JPress，比如是 `http://123.123.123.123/jpress`，就可以看到界面了

{% asset_img Snipaste_2019-02-26_10-03-56.png %}

按照提示一步一步操作即可。在连接 MySQL 时，注意一下 IP，不要写成 `localhost` 了，因为这里的 `localhost` 是只 JPress 容器自己的 `localhost`，而不是宿主机的 `localhost`

JPress 安装完成之后，执行 `docker restart <containerId>` 重启容器

# 创建一个 Java + Tomcat 镜像

## 下载 JDK 和 Tomcat

新建一个文件夹，比如 test

```
mkdir test
cd test
```

下载 JDK 并解压，得到一个名为 `jdk1.8.0_201` 的文件夹

```
wget --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz
tar -zxf jdk-8u201-linux-x64.tar.gz
```

下载 Tomcat 并解压，得到一个名为 `apache-tomcat-8.5.38` 的文件夹

```
wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.38/bin/apache-tomcat-8.5.38.tar.gz
tar -zxf apache-tomcat-8.5.38.tar.gz
```

## 编写 Dockerfile

```dockerfile
# 以 ubuntu:16.04 为基础镜像
FROM ubuntu:16.04

# 作者信息
LABEL maintainer="ikutarian46@ikutarian46.com"

# 拷贝 JDK 和 Tomcat 的安装包到指定位置
RUN mkdir -p /usr/local/soft
COPY jdk1.8.0_201 /usr/local/soft/jdk
COPY apache-tomcat-8.5.38 /usr/local/soft/tomcat

# 添加 JDK 和 Tomcat 的环境变量
ENV JAVA_HOME /usr/local/soft/jdk
ENV CATALINA_HOME /usr/local/soft/tomcat
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin

# 开放 8080 端口
EXPOSE 8080

# 容器启动时运行 Tomcat
CMD ["/usr/local/soft/tomcat/bin/catalina.sh", "run"]
```

## 构建

```
docker build -t ikutarian/tomcat:1.0 .
```

## 启动

```
docker run -d -p 8080:8080 ikutarian/tomcat:1.0
```

浏览器访问 `<宿主机ip>:8080` 即可看到界面了

# 一个使用阿里软件源的 Ubuntu 镜像

新建一个文件夹比如 `new_ubuntu`

准备好一个 Ubuntu 16.04 的阿里云源文件 `sources.list`， 内容为

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
```

阿里云软件源如何配置可以看这篇文章：《{% post_link ubuntu软件源 %}》

然后创建一个 Dockerfile 文件，填入以下内容

```dockerfile
# 以 ubuntu:16.04 为基础镜像
FROM ubuntu:16.04

# 备份原有的源
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak

# 复制阿里源
COPY sources.list /etc/apt/sources.list

# 更新源
RUN apt-get update
```

然后是构建

```
docker build -t ikutarian/new_ubuntu .
```

# 自己创建一个 Nginx 镜像

新建一个文件夹，比如 `static_web`，然后创建一个 Dockerfile 文件，填入如下内容

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

构建

```
docker build -t ikutarian/static_web .
```

启动容器

```
docker run -d -p 80:80 ikutarian/static_web
```

访问宿主机的 IP 就能看到 Nginx 的页面了

# 一个体积很小的 Docker 镜像

官方的 hello-world 镜像才 1.84KB，算是很小了。我也试试看能不能创建一个体积很小的 Docker 镜像

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        2 months ago        1.84kB
```

首先创建一个文件夹 `hello`，然后在文件夹中新建一个 `hello.c`，填入以下内容

```c
#include<stdio.h>

int main()
{
  printf("Hello, World\n");
  return 0;
}
```

然后编译 `hello.c`，生成可执行文件 `hello`

```
gcc hello.c -o hello
```

运行可执行文件 `hello`

```
./hello
```

就可以输出内容

```
Hello, World
```

编写 Dockerfile，填入如下内容

```dockerfile
# 使用空白镜像
FROM scratch

# 作者信息
LABEL maintainer="ikutarian <ikutarian@ikutarian.com>"

# 拷贝 hello 文件到根目录
COPY hello /

# 容器启动时运行的命令
CMD ["/hello"]
```

构建镜像，取名为 `ikutarian/hello`

```
docker build -t ikutarian/hello .
```

查看刚刚构建好的镜像，可以看到体积为 10.6KB，算是很小了

```
 docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ikutarian/hello     latest              3dfd813fabfd        53 seconds ago      10.6kB
hello-world         latest              fce289e99eb9        2 months ago        1.84kB
```

现在运行一下

```
docker run --rm ikutarian/hello
```

却发现报错了

```
standard_init_linux.go:190: exec user process caused "no such file or directory"
```

由于刚刚我们编译 `hello.c` 的时候，采用的是动态链接，gcc 帮我们默认链接了库文件，而 `ikutarian/hello` 中只有根目录下一个 `hello` 文件，并没有这些库文件，所以容器中的 `hello` 肯定运行不起来。使用 `ldd hello` 命令查看 `hello` 链接了哪些库文件

```
/lib/ld-musl-x86_64.so.1 (0x7f12fcfc8000)
libc.musl-x86_64.so.1 => /lib/ld-musl-x86_64.so.1 (0x7f12fcfc8000)
```

这次我们采用静态编译的方式编译 `hello.c` 

```
gcc -static hello.c -o hello
```

重新构建镜像，现在的镜像大小是 82.7kB，因为加入了库文件，所以镜像变大了

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ikutarian/hello     latest              cdd11ba57412        8 seconds ago       82.7kB
hello-world         latest              fce289e99eb9        2 months ago        1.84kB
```

不过现在重新运行镜像，没有问题，可以输出 `Hello, World`。不过和官方的 `hello-world` 镜像比起来，我的 `ikutarian/hello` 镜像体积太大了。到 Github 上  `hello-world` 的仓库]查看一下

Dockerfile 如下

```dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]
```

`hello.c` 如下

```c
//#include <unistd.h>
#include <sys/syscall.h>

#ifndef DOCKER_IMAGE
	#define DOCKER_IMAGE "hello-world"
#endif

#ifndef DOCKER_GREETING
	#define DOCKER_GREETING "Hello from Docker!"
#endif

#ifndef DOCKER_ARCH
	#define DOCKER_ARCH "amd64"
#endif

const char message[] =
	"\n"
	DOCKER_GREETING "\n"
	"This message shows that your installation appears to be working correctly.\n"
	"\n"
	"To generate this message, Docker took the following steps:\n"
	" 1. The Docker client contacted the Docker daemon.\n"
	" 2. The Docker daemon pulled the \"" DOCKER_IMAGE "\" image from the Docker Hub.\n"
	"    (" DOCKER_ARCH ")\n"
	" 3. The Docker daemon created a new container from that image which runs the\n"
	"    executable that produces the output you are currently reading.\n"
	" 4. The Docker daemon streamed that output to the Docker client, which sent it\n"
	"    to your terminal.\n"
	"\n"
	"To try something more ambitious, you can run an Ubuntu container with:\n"
	" $ docker run -it ubuntu bash\n"
	"\n"
	"Share images, automate workflows, and more with a free Docker ID:\n"
	" https://hub.docker.com/\n"
	"\n"
	"For more examples and ideas, visit:\n"
	" https://docs.docker.com/get-started/\n"
	"\n";

void _start() {
	//write(1, message, sizeof(message) - 1);
	syscall(SYS_write, 1, message, sizeof(message) - 1);

	//_exit(0);
	syscall(SYS_exit, 0);
}
```

Dockerfile 和我的是一样的，之所以 `hello-world` 之所以这么小，就是因为在 `hello.c` 上做的文章。C 语言我不熟，研究就到此为止吧

# Hexo 的持续集成