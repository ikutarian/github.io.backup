---
title: curl的安装与使用
date: 2019-01-10 15:32:30
tags:
  - curl
categories:
  - 安装与配置
---

# 是什么

最近在学习 Elasticsearch 的时候推荐使用 curl 命令来交互，于是就学习一下 curl

[curl 官网](https://curl.haxx.se/)上是这么描述的

> command line tool and library for transferring data with URLs

<!-- more -->

# 下载

我的平台是 Windows 10 64位，打开[网站](https://curl.haxx.se/download.html)，拉到最底部，选择这个

{% asset_img 20190110153832.png %}

然后选择这个下载

{% asset_img 20190110153935.png %}

可以看到下载的这个版本，功能很丰富，这些东西都有包括

{% asset_img 20190110154019.png %}

下载后会得到名为 `curl-7.63.0-win64-mingw.zip` 的压缩包，解压到某一个文件夹，比如 `E:\curl-7.63.0-win64-mingw` 里

# 加入环境变量

新建一个名为 `CURL_HOME` 的环境变量，内容为

```
E:\curl-7.63.0-win64-mingw
```

然后在 PATH 里新增一条

```
%CURL_HOME%\bin
```

这样就好了
