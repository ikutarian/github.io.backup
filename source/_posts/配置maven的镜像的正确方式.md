---
title: 配置Maven镜像的正确方式
date: 2020-02-12 16:19:07
tags:
  - Maven
  - 阿里云
  - 镜像
categories:
  - 安装与配置
---

因为网络的原因，Maven 默认的镜像加载速度很慢，所以需要使用镜像加快速度。网上关于配置镜像的文章很多，但是大部分都是错误的，甚至阿里云的官方文档也是错误的

<!-- more -->

网上经常能看到这样的配置

```xml
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>*</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

阿里云的官方文档也是这么写

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

其实吧，这是个错误的配置。问题在于 `<mirrorOf>*</mirrorOf>` 这段配置。它使用 `*` 通配符，说明了任何的 jar 包都去阿里云找。但是有些 jar 包，比如 Spring Cloud 的 jar 包在阿里云的 `public` 仓库里是找不到的。

正确的方式应该是这样。其中 `public` 仓库不用配置，而是应该配置 `central` 和 `jcenter`

```xml
<mirror>
  <id>aliyun-central</id>
  <mirrorOf>central</mirrorOf>
  <name>aliyun-central</name>
  <url>https://maven.aliyun.com/nexus/content/repositories/central</url>
</mirror>
<mirror>
  <id>aliyun-jcenter</id>
  <mirrorOf>jcenter</mirrorOf>
  <name>aliyun-jcenter</name>
  <url>https://maven.aliyun.com/nexus/content/repositories/jcenter</url>
</mirror>
<mirror>
  <id>aliyun-spring</id>
  <mirrorOf>spring</mirrorOf>
  <name>aliyun-spring</name>
  <url>https://maven.aliyun.com/nexus/content/repositories/spring</url>
</mirror>
<mirror>
  <id>aliyun-spring-plugin</id>
  <mirrorOf>spring-plugin</mirrorOf>
  <name>aliyun-spring-plugin</name>
  <url>https://maven.aliyun.com/nexus/content/repositories/spring-plugin</url>
</mirror>
```

不同的 `mirror` 应该配置不同的 `url` 才对。那么这些信息去那里找呢？打开阿里云的[官网](https://help.aliyun.com/document_detail/102512.html)，就可以看到这张表格

{% asset_img Snipaste_2020-02-12_16-26-50.png %}

仓库名称就是 `mirrorOf` 对应的名称。不同的 `mirror` 配置不同的 `url`，这样就不会错了