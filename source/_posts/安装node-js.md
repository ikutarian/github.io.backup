---
title: 安装node.js
date: 2018-09-14 17:16:39
tags:
  - node.js
categories:
  - 安装与配置
---

## 安装

### Windows下的安装

官网：https://nodejs.org/en/
官网上会列出两个版本的 node.js

{% asset_img 3617116-3c0b968fcf6d6690.png %}

一般来说选择 LTS 版本就好。下载安装包，双击安装。

<!-- more -->

### OSX 下的安装

最快、最简便的方法是不用去官网下载安装包了，使用 [Homebrew](http://brew.sh/) 安装就好

### Linux 下的安装

通过 `yum|apt-get` 命令安装

### 查看 node.js 的版本

安装成功后，在终端下输入

```
node -v
```

可以查看 node 的版本。

输入

```
npm -v
```

可以查看 npm 的版本

## 切换npm源

用过 ubuntu 的可能会明白，国外的源访问很慢，我们都会把源地址切换到国内的镜像。nrm 同理，用来切换官方 npm 源和国内的 npm 源。

有两种方式实现：

* 安装npm源管理工具[nrm](https://github.com/Pana/nrm)
* 直接将本地的npm仓库指向淘宝的镜像地址

### 安装npm源管理工具

[nrm](https://github.com/Pana/nrm) 是一个管理 npm 源的工具

#### 全局安装 nrm

```
npm i nrm -g
```

#### 查看当前 nrm 内置的几个 npm 源的地址：

命令

```
nrm ls
```

输出结果

```
C:\Users\Administrator>nrm ls

* npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
  taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/


C:\Users\Administrator>
```

#### 切换到淘宝源

命令

```
nrm use taobao
```

输出结果

```
C:\Users\Administrator>nrm use taobao
                         verb config Skipping project config: C:\Users\Administ

   Registry has been set to: https://registry.npm.taobao.org/


C:\Users\Administrator>nrm ls

  npm ---- https://registry.npmjs.org/
  cnpm --- http://r.cnpmjs.org/
* taobao - https://registry.npm.taobao.org/
  nj ----- https://registry.nodejitsu.com/
  rednpm - http://registry.mirror.cqupt.edu.cn/
  npmMirror  https://skimdb.npmjs.com/registry/
  edunpm - http://registry.enpmjs.org/


C:\Users\Administrator>
```

这样淘宝源就切换完成了。

### 直接将本地的npm仓库指向淘宝的镜像地址

```
npm config set registry https://registry.npm.taobao.org
```

配置后可通过下面方式来验证是否成功

```
npm config get registry
```