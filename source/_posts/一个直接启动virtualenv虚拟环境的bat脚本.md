---
title: 一个直接启动virtualenv虚拟环境的bat脚本
date: 2018-09-17 10:46:00
tags:
  - virtualenv
  - Python
  - bat
  - 脚本
categories:
  - Python
---

在 Windows 系统中，每次启动 virtualenv 时，我都要输入

```
cd ENV\Scripts\
activate
```

启动虚拟环境之后，还要再回到工程目录中

```
cd ../..
```

太麻烦了。如果有一个 bat 脚本，直接启动 virtualenv 虚拟环境，然后进入工程目录，那就很方便了。

<!-- more -->

经过查找资料，写出了这个脚本

```
@echo off
call ENV\Scripts\activate.bat
cmd .
```

脚本很简单。鼠标双击运行，就可以打开控制台，启动 virtualenv 虚拟环境，进入工程目录。