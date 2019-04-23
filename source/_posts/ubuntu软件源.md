---
title: ubuntu软件源
date: 2019-02-27 15:36:54
tags:
  - Ubuntu
  - 软件源
  - 阿里云
categories:
  - Linux
---

Ubuntu 原生的软件源在国内访问太慢了，如果使用国内的镜像的话，访问速度会好一点

<!-- more -->

# 备份源文件

首先备份一下，如果出错了还可以恢复

```
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

# 更换源的方法

有两种方法：

1. 通过 sed 命令进行文本替换
2. 替换 /etc/apt/sources.list 文件

# 通过 sed 命令进行文本替换

一般情况下，更改 `/etc/apt/sources.list` 文件中 Ubuntu 默认的源地址 `http://archive.ubuntu.com/` 为你想要的地址即可，比如我要把源更换成中科大的

```
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```

通过这条命令，就可以把 `/etc/apt/sources.list` 文件中 Ubuntu 默认的源地址给替换了。接着还要 update 一下

```
apt-get update
```

第二种方法就是手动替换 `/etc/apt/sources.list` 文件了，继续看下面的文档吧

# 替换 /etc/apt/sources.list 文件

## 软件源怎么找

打开[清华大学的软件源首页](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)就能看到了，列出了不同版本的 Ubuntu 使用的源

{% asset_img Snipaste_2019-02-27_15-57-04.png %}

可以输入 `lsb_release -a` 查看一下自己 Ubuntu 的版本，然后根据版本选择源即可

清华大学的软件源是教育网的，有时候还会不稳定，还是用阿里云的好了

打开[阿里云的镜像站的说明文档](https://opsx.alibaba.com/guide?lang=zh-CN&document=699f2968-801e-11e8-9e1d-00163e04cdbb)

这里说了如果是互联网用户，用 `mirrors.aliyun.com` 这个域名

{% asset_img Snipaste_2019-02-27_16-00-32.png %}

我们可以把清华大学的源拿过来，把 URL 改一下就可以变成阿里云的源了

比如这个是 16.04 版本的源

```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
```

可以使用记事本的替换功能，把 `mirrors.tuna.tsinghua.edu.cn` 替换成 `mirrors.aliyun.com`，这样就是阿里云的源了

## 替换源

把 `/etc/apt/sources.list` 内容替换成如下

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

## 刷新

输入以下命令

```
apt-get update
```

# 源的规律

我发现不同版本 Ubuntu 的软件源都差不多，比如下面是 16.04 的其中一个源

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
```

这是 18.04 的

```
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
```

区别在于版本名称（codename）不同，一个是 `xenial` 一个是 `bionic`。输入 `lsb_release -a` 就可以看到 codename 的信息，比如我自己的

```
root@ikutarian:~/docker/static_web# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.5 LTS
Release:	16.04
Codename:	xenial
```

可以看到版本（Release）是 16.04，版本名称（Codename）是 `xenial`。所以不同版本的源，只需要把 Codename 换一下其他部分不用变就可以拿来使用了

# 更简单的方法

打开[阿里云镜像站](https://opsx.alibaba.com/mirror)，可以看到一堆的镜像列表，点击右边的“帮助”

{% asset_img Snipaste_2019-04-09_16-45-11.png %}

会显示一个弹窗，详细说明了更换镜像的方法了，按照操作的来就行

{% asset_img Snipaste_2019-04-09_16-46-02.png %}

# 参考

- [Ubuntu镜像使用帮助](https://lug.ustc.edu.cn/wiki/mirrors/help/ubuntu)