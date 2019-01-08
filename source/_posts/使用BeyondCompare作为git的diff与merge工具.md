---
title: 使用BeyondCompare作为git的diff与merge工具
date: 2018-10-24 15:00:58
tags:
  - Git
  - BeyondCompare
  - diff
  - merge
categories:
  - 安装与配置
---

Beyond Compare 是一个图形化的文档比较工具，使用起来很方便。而且，在使用 Git 的过程中，感觉使用命令行进行代码比较与合并不是很方便，所以就决定 Git 搭配 Beyond Compare 使用

<!-- more -->

## Beyond Compare 的安装

官网：http://www.scootersoftware.com/
密钥：

```
w4G-in5u3SH75RoB3VZIX8htiZgw4ELilwvPcHAIQWfwfXv5n0IHDp5hv
1BM3+H1XygMtiE0-JBgacjE9tz33sIh542EmsGs1yg638UxVfmWqNLqu-
Zw91XxNEiZF7DC7-iV1XbSfsgxI8Tvqr-ZMTxlGCJU+2YLveAc-YXs8ci
RTtssts7leEbJ979H5v+G0sw-FwP9bjvE4GCJ8oj+jtlp7wFmpVdzovEh
v5Vg3dMqhqTiQHKfmHjYbb0o5OUxq0jOWxg5NKim9dhCVF+avO6mDeRNc
OYpl7BatIcd6tsiwdhHKRnyGshyVEjSgRCRY11IgyvdRPnbW8UOVULuTE
```

目前最新的版本是 4.x。我把软件安装到了 `E:\BeyondCompare4` 下

## 配置 Git

在 Beyond Compare 的官网，有对 Git 的配置进行说明的[文档](http://www.scootersoftware.com/support.php?c=kb_vcs.php)，可以查看“GIT FOR WINDOWS”这个小节

首先确定一下 Git 的版本，因为文档说了对于不同的 Git 版本有不同的配置方法

控制台中输入

```
git --version
```

即可得到 Git 的版本号

```
git version 2.19.1.windows.1
```

## 配置diff

现在就可以进行配置了，打开控制台，输入

```
git config --global diff.tool bc
git config --global difftool.bc.path "e:/BeyondCompare4/bcomp.exe"
git config --global difftool.prompt false
```

要使用diff，只需要在控制台中输入

```
git difftool foofile.txt
```

## 配置merge

打开控制台，输入

```
git config --global merge.tool bc
git config --global mergetool.bc.path "e:/BeyondCompare4/bcomp.exe"
git config --global mergetool.prompt false
```

要使用merge，只需要在控制台中输入

```
git mergetool foofile.txt
```

## 额外说明

虽然 Beyond Compare 的说明文档没有要求配置

```
git config --global difftool.prompt false
git config --global mergetool.prompt false
```

这是因为在输入 `git difftool foofile.txt` 或者 `git mergetool foofile.txt` 时，Git 会提示如下信息

```
$ git difftool

Viewing (1/1): 'src/main/java/com/ikutarian/mmall/service/impl/CategoryServiceImpl.java'
Launch 'bc' [Y/n]? 
```

我们还需要手动输入 `y` 或者 `n`。所以配置了 `prompt` 为 `false` 直接就进行文件diff和merge，不需要询问用户

## 参考

- [BCompare4注册码](https://blog.csdn.net/qq_15204179/article/details/81907628)
- [git 使用beyond compare来diff与merge](https://blog.csdn.net/yu12377/article/details/73526751)
- [Git下使用Beyond Compare作为比较和合并工具](https://blog.csdn.net/wangkai_123456/article/details/76704586)