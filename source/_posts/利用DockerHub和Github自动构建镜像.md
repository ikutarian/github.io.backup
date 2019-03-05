---
title: 利用DockerHub和Github自动构建镜像
date: 2019-03-01 10:25:30
tags:
  - DockerHub
  - Docker
  - Github
  - Dockerfile
categories:
  - Docker
---

本地写好一份 Dockerfile，然后上传到 Github。DockerHub 会根据上传的 Dockerfile 自动构建镜像

<!-- more -->

我写好一份 Dockerfile，然后上传到 Github 的[仓库](https://github.com/ikutarian/docker_repo)，目录结构是这样

```
├─ README.md
│
└─ aliyun_ubuntu/
    ├─  Dockerfile
    └─ sources.list
```

然后在 DockerHub 新建一个 Repository，取名为 `ikutarian/aliyun_ubuntu`，并创建构建脚本

{% asset_img Snipaste_2019-03-01_10-34-26.png %}

根据 Github 仓库的目录结构，Dockerfile 在 `aliyun_ubuntu` 文件夹里， 所以 `Build Context` 要写成 `/aliyun_ubuntu/`。至于 `Dockerfile location` 的值，按照文档的说法

> Path from the repository root to the files to build. The Dockerfile location is relative to this path

也就是说 `Dockerfile location` 是相对 `Build Context` 路径来查找的。如果你的 Dockerfile 文件名没有改的话，默认就写 `Dockerfile`

填写完毕之后就开始构建

可以查看构建的 Log，比如我的

```
Building in Docker Cloud's infrastructure...
Cloning into '.'...
Warning: Permanently added the RSA host key for IP address '192.30.253.112' to the list of known hosts.
Reset branch 'master'
Your branch is up-to-date with 'origin/master'.
KernelVersion: 4.4.0-1060-aws
Components: [{u'Version': u'18.03.1-ee-3', u'Name': u'Engine', u'Details': {u'KernelVersion': u'4.4.0-1060-aws', u'Os': u'linux', u'BuildTime': u'2018-08-30T18:42:30.000000000+00:00', u'ApiVersion': u'1.37', u'MinAPIVersion': u'1.12', u'GitCommit': u'b9a5c95', u'Arch': u'amd64', u'Experimental': u'false', u'GoVersion': u'go1.10.2'}}]
Arch: amd64
BuildTime: 2018-08-30T18:42:30.000000000+00:00
ApiVersion: 1.37
Platform: {u'Name': u''}
Version: 18.03.1-ee-3
MinAPIVersion: 1.12
GitCommit: b9a5c95
Os: linux
GoVersion: go1.10.2
Starting build of index.docker.io/ikutarian/aliyun_ubuntu:latest...
Step 1/4 : FROM ubuntu:16.04
---> 7e87e2b3bf7a
Step 2/4 : RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
---> Running in b2aa14f7a9c4
Removing intermediate container b2aa14f7a9c4
---> f11911c81be9
Step 3/4 : COPY sources.list /etc/apt/sources.list
---> 7b450ef519d8
Step 4/4 : RUN apt-get update
---> Running in da9e1409463f
Get:1 http://mirrors.aliyun.com/ubuntu xenial InRelease [247 kB]
Get:2 http://mirrors.aliyun.com/ubuntu xenial-updates InRelease [109 kB]
Get:3 http://mirrors.aliyun.com/ubuntu xenial-backports InRelease [107 kB]
Get:4 http://mirrors.aliyun.com/ubuntu xenial-security InRelease [109 kB]
Get:5 http://mirrors.aliyun.com/ubuntu xenial/main amd64 Packages [1558 kB]
Get:6 http://mirrors.aliyun.com/ubuntu xenial/restricted amd64 Packages [14.1 kB]
Get:7 http://mirrors.aliyun.com/ubuntu xenial/universe amd64 Packages [9827 kB]
Get:8 http://mirrors.aliyun.com/ubuntu xenial/multiverse amd64 Packages [176 kB]
Get:9 http://mirrors.aliyun.com/ubuntu xenial-updates/main amd64 Packages [1180 kB]
Get:10 http://mirrors.aliyun.com/ubuntu xenial-updates/restricted amd64 Packages [13.1 kB]
Get:11 http://mirrors.aliyun.com/ubuntu xenial-updates/universe amd64 Packages [944 kB]
Get:12 http://mirrors.aliyun.com/ubuntu xenial-updates/multiverse amd64 Packages [19.1 kB]
Get:13 http://mirrors.aliyun.com/ubuntu xenial-backports/main amd64 Packages [7942 B]
Get:14 http://mirrors.aliyun.com/ubuntu xenial-backports/universe amd64 Packages [8532 B]
Get:15 http://mirrors.aliyun.com/ubuntu xenial-security/main amd64 Packages [786 kB]
Get:16 http://mirrors.aliyun.com/ubuntu xenial-security/restricted amd64 Packages [12.7 kB]
Get:17 http://mirrors.aliyun.com/ubuntu xenial-security/universe amd64 Packages [541 kB]
Get:18 http://mirrors.aliyun.com/ubuntu xenial-security/multiverse amd64 Packages [6116 B]
Fetched 15.7 MB in 9s (1732 kB/s)
Reading package lists...
Removing intermediate container da9e1409463f
---> b6493b2ebb97
Successfully built b6493b2ebb97
Successfully tagged ikutarian/aliyun_ubuntu:latest
Pushing index.docker.io/ikutarian/aliyun_ubuntu:latest...
Done!
Build finished
```

可以看到构建已经成功了