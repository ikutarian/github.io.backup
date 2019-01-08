---
title: 导入Oracle的JDBC驱动到本地Maven仓库
date: 2018-12-12 21:22:13
tags:
  - Maven  
  - Oracle
  - JDBC
categories:
  - 安装与配置
---

Oracle 的 JDBC Driver 没有放到公共 Maven 仓库里，需要自己下载

<!-- more -->

## 下载地址

首先确认一下自己的 Oracle 版本，我的是 Oracle 11g2。

如果不知道本地 Oracle 的版本可以通过：`SELECT * FROM v$version;` 查询。结果如下所示：

{% asset_img QQ截图20181213095704.png %}

百度一下，找到了[这个地址](https://www.oracle.com/technetwork/database/enterprise-edition/jdbc-112010-090769.html)。而且 JDK 的版本是 1.8，所以用这个就行

{% asset_img QQ图片20181213095401.png %}

懒得去官网下载，可以[本地下载](/uploads/ojdbc6.jar)

## 安装到本地 Maven 仓库

既然公共 Maven 仓库找不到，那就放到本地仓库好了。使用以下命令

```
mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.1.0 -Dpackaging=jar -Dfile=F:\ojdbc6.jar
```

安装成功，会输出

```
c:\>mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.1.0 -Dpackaging=jar -Dfile=F:\ojdbc6.jar
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< org.apache.maven:standalone-pom >-------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom ---
[INFO] Installing F:\ojdbc6.jar to E:\Maven_Repo\com\oracle\ojdbc6\11.2.0.1.0\ojdbc6-11.2.0.1.0.jar
[INFO] Installing C:\Users\Think\AppData\Local\Temp\mvninstall8681601283517429280.pom to E:\Maven_Repo\com\oracle\ojdbc6\11.2.0.1.0\ojdbc6-11.2.0.1.0.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.957 s
[INFO] Finished at: 2018-12-12T10:57:29+08:00
[INFO] ------------------------------------------------------------------------
```

## 使用

在 pom.xml 中添加依赖

```
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0.1.0</version>
</dependency>
```

需要注意的是：上述xml文件中的 `groupId`、`artifactId` 和 `version` 必须和使用 “mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc6 -Dversion=11.2.0.1.0 -Dpackaging=jar -Dfile=F:\ojdbc6.jar” 中的一致。