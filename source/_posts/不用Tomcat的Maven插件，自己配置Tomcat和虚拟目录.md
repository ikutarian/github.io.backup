---
title: 不用 Tomcat 的 Maven 插件，自己配置 Tomcat 和虚拟目录
date: 2018-09-14 16:56:54
tags:
  - Tomcat
  - Maven
  - IDEA
categories:
  - 安装与配置
---

以前一直使用 `tomcat7-maven-plugin` 插件，虽然能直接跑起来项目，不用去关心如何配置 tomcat。但是出来混总是要还的，现在就必须得用原生的 Tomcat 了。因此记录过程，毕竟好记性不如烂笔头。

<!-- more -->

## 配置 Tomcat

下载好 `apache-tomcat-8.0.39.zip` 之后，解压。比如解压到 `E:\apache-tomcat-8.0.39` 中。

然后就是在 IDEA 中配置 Tomcat 了。

首先在右上角选择 Edit Configuration

{% asset_img 3617116-34600d3ae701136d.png %}

然后选择本地的 Tomcat

{% asset_img 3617116-a9224854efcf7374.png %}

点击这里，配置好 Tomcat 的位置

{% asset_img 3617116-b48d232f735e4559.png %}

{% asset_img 3617116-732066c7f5f6f024.png %}

给 Tomcat 取个名字。把项目的 war 包加入

{% asset_img 3617116-82e8b3c786cdda97.png %}

{% asset_img 3617116-cc4ebc4c9f9a1ba2.png %}

同时，写上 base url

{% asset_img 3617116-5ec2e5d4c43c4ffc.png %}

启动 Tomcat，访问 http://localhost/blog 就可以了

## 虚拟目录

和之前添加 war 包一样的步骤，不过一个不同

选择 External Source

{% asset_img 3617116-a179502e3caea0f6.png %}

然后选择要映射的路径

{% asset_img 3617116-1a3352cc4aed9eea.png %}

然后写上 base url

{% asset_img 3617116-93eb5e2c5d5e5fe1.png %}

比如访问 `E:\upload\2018\06\11\ec28dcdc-2a2f-44c4-80da-6bfa756060bc.jpg` 图片文件，只需要输入 `http://localhost:8080/upload/2018/06/11/ec28dcdc-2a2f-44c4-80da-6bfa756060bc.jpg` 即可