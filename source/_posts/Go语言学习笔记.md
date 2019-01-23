---
title: Go语言学习笔记
date: 2019-01-09 10:35:22
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

# Hello World

新建一个名为 `hello.go` 的文件，内容为

```go
package main

import "fmt"

func main() {
    fmt.Printf("hello, go")
}
```

然后在 `hello.go` 的文件所在的路径运行如下命令

```
go run hello.go
```

就可以输出

```
hello go
```

# Workspace

所有的代码都存放在一个 workspace 里。它其实就是一个目录，下面有两个子目录：

- src：存放 Go 源码文件
- bin：存放可执行的文件

go tool 会编译 src 里的源码，然后安装二进制文件到 bin 目录里

# GOPATH

GOPATH 环境变量指定了 workspace 位置

## Windows 下的设置

打开环境变量，可以发现，默认在“用户环境变量”中有一个 GOPATH

{% asset_img Snipaste_2019-01-09_10-17-10.png %}

删掉吧，没用。改在“系统环境变量”中添加一个 GOPATH（路径不要和 Go 安装路径一样）

{% asset_img Snipaste_2019-01-09_10-19-01.png %}

在命令行中输入 `go env` 查看 go 的环境变量配置信息，就可以看到 GOPATH 的值了

{% asset_img Snipaste_2019-01-09_10-21-12.png %}

或者直接输入 `go env GOPATH` 也行

```
C:\Users\Think>go env GOPATH
F:\go
```

为了方便，把 `%GOPATH%\bin` 也加入环境变量中

# 打印函数

Go 的 fmt 包下有两个打印内容的方法：

- Println
- Printf

`Println` 可以打印任何类型的变量。`Printf` 必须使用格式化输出才能正确调用

```
a := 10
fmt.Println(a)　　    //right
fmt.Println("abc")　　//right
fmt.Printf("%d",a)　　//right
fmt.Printf(a)　　     //error
```