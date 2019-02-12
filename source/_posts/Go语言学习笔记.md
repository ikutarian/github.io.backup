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

# 起步

## Go 语言的特点

1. 静态类型、编译型的开源语言
2. 脚本话的语法，支持多种编程范式（函数式&面向对象）
3. 原生、给力的并发编程支持（注意：原生支持和函数库支持的区别）

## Go 语言的优势与劣势

### 优势

1. 脚本化的语法
2. 静态类型 + 编译型，程序运行速度有保障
3. 原生地支持并发编程

### 劣势

1. 语法糖没有 Python 和 Ruby 那么多（我反而觉得这是优势。因为根据 2/8原则，开发时间占 2，维护时间占 8，语法糖太多代码看起来都吃力）
2. 目前程序的运行速度不及 C
3. 第三方库不够多

## 安装

[官网](https://golang.org/dl/)可以下载三个平台的安装包

1. [Microsoft Windows](https://dl.google.com/go/go1.11.4.windows-amd64.msi)
2. [Apple macOS](https://dl.google.com/go/go1.11.4.darwin-amd64.pkg)
3. [Linux](https://dl.google.com/go/go1.11.4.linux-amd64.tar.gz)

### Windows 下的安装

下载好安装包之后，把 Go 安装到某一个路径，比如 `E:\Go`，安装包会自动把 `E:\Go\bin` 加入环境变量 PATH 中

在命令行验证一下，输入

```
go version
```

有打印出版本信息就说明安装成功了

```
go version go1.11.4 windows/amd64
```

### Linux 下的安装

#### 下载

```
wget https://studygolang.com/dl/golang/go1.11.linux-amd64.tar.gz
```

#### 解压到 `usr/local` 文件夹

```
tar zxf go1.11.linux-amd64.tar.gz -C /usr/local
```

#### 配置环境变量

打开 `/etc/profile` 文件，在末尾**追加**如下值

```
export PATH=$PATH:$usr/local/go/bin;$GOBIN
```

#### 验证安装结果

在命令行任意路径输入

```
go version
```

看看是否能打印出如下版本信息

```
go version go1.11 linux/amd64
```

# 环境变量

## Workspace

使用 Go 语言进行开发，代码都是存放在 workspace 里。它其实就是一个目录，下面有两个子目录：

- src：存放 Go 源码文件。这些源码文件会被组织成一个 package，src 里的每个子目录都是一个 package
- bin：存放可执行的文件。go tool 会编译 src 里的源码，然后安装二进制文件到 bin 目录里

## GOPATH

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

为了方便，把 `%GOPATH%\bin` 也加入环境变量中，这样我们就可以直接执行编译后的 Go 程序了

## Hello World

在 `%GO_PATH%\github.com\[github昵称]\hello` 文件夹中新建一个名为 `main.go` 的文件，内容为

```go
package main

import "fmt"

func main() {
    fmt.Printf("hello, go")
}
```

然后在 `main.go` 的文件所在的路径运行如下命令

```
go run main.go
```

就可以输出

```
hello go
```

## go build 与 go run

`go build` 是编译操作。会生成一个可执行文件，windows 下文件后缀是 `.exe`，运行 `.exe` 文件就可以看到运行结果。

`go run` 把编译和运行合并起来，直接就可以看到运行结果

开发的时候用 `go run`，发布时用 `go build`

# 数据类型

数据类型分为动态类型和静态类型。在 Go 中，可以显式地声明类型，也可以不写，让编译器自己去推断

- bool
- int
- float
- string
- array

## 检测变量的类型

利用 `reflect` 包的 `TypeOf()` 方法，比如：

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var a = 123
    fmt.Println(reflect.TypeOf(a))
}
```

## 类型转换

利用 `strconv` 包的方法，可以将基本类型与字符串进行互相转换

# 变量

值的引用

## 定义变量的方式

```go
// 初始化和赋值写在一起
var s string = "hello world"

// 只使用 var 关键字，不指定变量类型
var s = "hello world"

// var 关键字和变量类型都不写（只能在函数内使用）
s := "hello world"

// 先初始化，后赋值
var s
s = "hello world"

// 定义多个相同类型的变量
var s, t string = "foo", "bar"

// 定义多个不同类型的变量
var (
    s = "foo"
    i = 4
)
```

推荐使用这两种：

```
// 只使用 var 关键字，不指定变量类型
var s = "hello world"

// var 关键字和变量类型都不写（只能在函数内使用）
s := "hello world"
```

## 0 值

一个变量被定义，如果没有给它赋值的话，默认是 0 值

|类型|0 值|
|int|0|
|float|0|
|bool|false|
|string|""|

**注意**

> string 类型变量的 0 值是 `""`，而不是 `nil`。Go 不允许在变量初始化时使用 `nil` 进行赋值

## 变量的范围

`{}` 包括起来的内容叫 block

block 可以嵌套

内部的 block 可以访问外部的变量，外部的 block 无法访问内部 block 的变量

## 指针

变量的内存地址就是指针。所谓的内存地址，可以把内存想象成一排房间，每个房间都有一个门牌号，变量的值就是房间里住的人

可以通过在变量名前面加一个 `&` 获得内存地址

```go
var s = "Hello World"
fmt.Println(&s)  
```

### 变量在函数之间传递

一个变量传递给函数，会重新开辟一个内存空间，把原来的变量值拷贝一份存放到新的内存空间里

```go
func showMemoryAddress(x int) {
    fmt.Println(&x)
}

func main() {
    i := 1
    fmt.Println(&i)
    showMemoryAddress(i)
}
```

输出

```
0xc000058058
0xc000058090
```

可以看到内存地址变化了

如果把变量的内存地址拿来在函数之间进行传递的话，这样即使拷贝也是拷贝一份内存地址，这个内存地址存储的值还是一样的

```go
func showMemoryAddress(x *int) {
    fmt.Println(x)
}

func main() {
    i := 1
    fmt.Println(&i)
    showMemoryAddress(&i)
}
```

现在方法 `showMemoryAddress` 的参数类型从 `int` 变成了 `*int`，表示参数是一个 int 类型的指针

通过 `*` 可以得到内存地址指向的变量的值

```go
func showMemoryAddress(x *int) {
    fmt.Println(*x)
}
```

现在将会打印

```
0xc000058058
1
```

## 常量

使用关键字 `const` 定义常量，比如

```
const greeting = "hello world"
```

# 函数