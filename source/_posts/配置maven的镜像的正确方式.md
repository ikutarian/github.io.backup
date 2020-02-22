---
title: 配置Maven镜像的正确方式
date: 2020-02-12 16:19:07
tags:
  - Maven
  - 阿里云
  - 镜像
  - lastUpdated
  - bat
  - 脚本
categories:
  - 安装与配置
---

因为网络的原因，Maven 默认的镜像加载速度很慢，所以需要使用镜像加快速度。网上关于配置镜像的文章很多，但是大部分都是错误的，甚至阿里云的官方文档也是错误的

<!-- more -->

# 错误的做法

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

# 正确的做法 1.0

正确的方式应该是这样。其中 `public` 仓库不用配置，而是应该配置 `central` 和 `jcenter`

```xml
<mirror>
  <id>aliyun-central</id>
  <mirrorOf>central</mirrorOf>
  <name>aliyun-central</name>
  <url>	https://maven.aliyun.com/repository/central</url>
</mirror>
<mirror>
  <id>aliyun-jcenter</id>
  <mirrorOf>jcenter</mirrorOf>
  <name>aliyun-jcenter</name>
  <url>https://maven.aliyun.com/repository/jcenter</url>
</mirror>
<mirror>
  <id>aliyun-spring</id>
  <mirrorOf>spring</mirrorOf>
  <name>aliyun-spring</name>
  <url>https://maven.aliyun.com/repository/spring</url>
</mirror>
<mirror>
  <id>aliyun-spring-plugin</id>
  < mirrorOf>spring-plugin</mirrorOf>
  <name>aliyun-spring-plugin</name>
  <url>https://maven.aliyun.com/repository/spring-plugin</url>
</mirror>
```

不同的 `mirror` 应该配置不同的 `url` 才对。那么这些信息去那里找呢？打开阿里云的[官网](https://help.aliyun.com/document_detail/102512.html)，就可以看到这张表格

{% asset_img Snipaste_2020-02-12_16-26-50.png %}

仓库名称就是 `mirrorOf` 对应的名称。不同的 `mirror` 配置不同的 `url`，这样就不会错了

# 正确的做法 2.0

经过几天的使用，发现阿里云真的坑。比如这个依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.12.RELEASE</version>
    <relativePath/>
</parent>
```

是从 central mirror 拉取的。但是阿里云的 central mirror 却没有！在阿里云这个依赖是放在 jcenter 里的

{% asset_img Snipaste_2020-02-14_16-14-25.png %}

所以镜像配置还得改。central 和 jcenter 都用阿里云的 public

```xml
<mirror>
  <id>aliyun-jcenter</id>
  <mirrorOf>jcenter</mirrorOf>
  <name>aliyun-jcenter</name>
  <url>https://maven.aliyun.com/repository/public</url> <!-- 改用 public -->
</mirror>
<mirror>
  <id>aliyun-central</id>
  <mirrorOf>central</mirrorOf>
  <name>aliyun-central</name>
  <url>https://maven.aliyun.com/repository/public</url> <!-- 改用 public -->
</mirror>
<mirror>
  <id>aliyun-spring</id>
  <mirrorOf>spring</mirrorOf>
  <name>aliyun-spring</name>
  <url>https://maven.aliyun.com/repository/spring</url>
</mirror>
<mirror>
  <id>aliyun-spring-plugin</id>
  <mirrorOf>spring-plugin</mirrorOf>
  <name>aliyun-spring-plugin</name>
  <url>https://maven.aliyun.com/repository/spring-plugin</url>
</mirror>
</mirrors>
```

# Maven仓库的垃圾文件清理

有时候因为网络的原因，Maven 依赖出现下载失败的情况，会在本地仓库的文件夹中生成一些以 `.lastUpdated` 为后缀的文件。可以使用脚本把它们删掉，然后再重试下载依赖

把下面的文本保存为 `.bat` 后缀的脚本，然后鼠标双击运行即可

```bat
@echo off
rem 指定仓库的位置
set REPOSITORY_PATH=E:\apache-maven-3.6.3_repo
rem 开始删除... 
for /f "delims=" %%i in ('dir /b /s "%REPOSITORY_PATH%\*lastUpdated*"') do (
    del /s /q %%i
)
rem 删除完成!!
pause
```