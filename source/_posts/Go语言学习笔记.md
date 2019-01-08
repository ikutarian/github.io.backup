---
title: 《Go语言第一课》笔记
date: 2019-01-05 15:36:05
tags:
  - Go
  - 笔记
categories:
  - Go
---

目前学习 Go 语言的笔记

<!-- more -->

# Go 语言的特点

1. 静态类型、编译型的开源语言
2. 脚本话的语法，支持多种编程范式（函数式&面向对象）
3. 原生、给力的并发编程支持（注意：原生支持和函数库支持的区别）

# Go 语言的优势与劣势

## 优势

1. 脚本化的语法
2. 静态类型 + 编译型，程序运行速度有保障
3. 原生地支持并发编程

## 劣势

1. 语法糖没有 Python 和 Ruby 那么多（我反而觉得这是优势。因为根据 2/8原则，开发时间占 2，维护时间占 8，语法糖太多代码看起来都吃力）
2. 目前程序的运行速度不及 C
3. 第三方库不够多

# 安装

[官网](https://golang.org/dl/)可以下载三个平台的安装包

1. [Microsoft Windows](https://dl.google.com/go/go1.11.4.windows-amd64.msi)
2. [Apple macOS](https://dl.google.com/go/go1.11.4.darwin-amd64.pkg)
3. [Linux](https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz)

## Windows 下的安装

下载好安装包之后，把 Go 安装到某一个路径，比如 `E:\Go`，安装包会自动把 `E:\Go\bin` 加入环境变量 PATH 中

在命令行验证一下，输入

```
go version
```

有打印出版本信息就说明安装成功了

```
go version go1.11.4 windows/amd64
```

## Linux 下的安装

### 下载

```
wget https://studygolang.com/dl/golang/go1.11.linux-amd64.tar.gz
```

### 解压到 `usr/local` 文件夹

```
tar zxf go1.11.linux-amd64.tar.gz -C /usr/local
```

### 配置环境变量

打开 `/etc/profile` 文件，在末尾**追加**如下值

```
export PATH=$PATH:$usr/local/go/bin;$GOBIN
```

### 验证安装结果

在命令行任意路径输入

```
go version
```

看看是否能打印出如下版本信息

```
go version go1.11 linux/amd64
```

# 怎么写代码

官网有一篇[文章](https://golang.org/doc/code.html)讲解了，现在总结一下

## 约定

- 所有的代码都存放在一个 workspace 里
- workspace 里有很多的版本控制工具管理的代码仓库
- 每个代码仓库有一个或多个的 package
- 每个 package 是一个目录，里面存放 Go 的源码文件
- package 的路径表示 import path

## Workspace

workspace 下有两个目录：

- src：存放 Go 源码文件
- bin：存放可执行的文件