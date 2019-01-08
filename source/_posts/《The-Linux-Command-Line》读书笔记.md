---
title: 《The Linux Command Line》读书笔记
date: 2018-11-08 09:14:44
tags:
  - Linux
  - 命令行
categories:
  - 读书笔记
---

读书计划开始了，这是我要阅读的第二本书，命令行操作还是很重要的

<!-- more -->

# 2 什么是 Shell

## root@ikutarian:~# 是什么

进入 bash，可以看到已经有一行文字显示在上面了

```
root@ikutarian:~#
```

- `root` 表示当前登陆的用户，我现在是以 root 用户登陆，所以显示的是 `root`
- `ikutarian` 是我这台电脑的名称
- `~` 表示当前所在的路径，目前是在 root 用户的 home 目录下
- `#` 表示当前是超级管理员角色。如果是普通用户，显示的是 `$`

## 命令历史记录

可以通过键盘 `↑`与键盘 `↓` 查看命令历史记录

## 几个常用的命令：

退出终端

```
exit
```

关机

```
shutdown now
```

查看当前日期

```
date
```

查看本月日历

```
cal
```

查看硬盘使用量

```
df -h
```

查看内存使用量

```
free -h
```

# 3 文件系统

## 文件系统的属性结构

Windows 是每个存储设备都有一个自己的文件系统，比如 C 盘、D 盘等。但是 Linux 不一样。不管有多少个存储设备，都是只有一个文件系统。这些存储设备是挂载到目录树上的

目录树只有一个根节点 `/`，然后下面文件和子目录，子目录里面又有文件和子目录

{% asset_img Snipaste_2018-11-09_21-33-27.png %}

## 查看当前的工作目录

```
pwd
```

pwd 就是 print working directory 的缩写

## home 目录

每个用户都有自己的 home 目录，比如 root 用户的 home 目录是 `/root`。一个叫 ikutarian 的用户的 home 目录是 `/home/ikutarian`

可以用 `~` 表示自己的 home 目录，输入 `cd ~` 即可跳转到自己的 home 目录

## 列出目录内容

```
ls
```

## 目录跳转

```
cd
```

有两种方式可以跳转：

1. 绝对路径 - `/path`
2. 相对路径 - `.` 表示当前目录，`..` 表示上一级目录

## 文件名的规范

1. 以 `.` 开头的是隐藏文件
2. 文件名大小写敏感
3. Linux 没有文件扩展名的概念
4. 文件名不要带空格，可以用 `_` 代替

# 4 探究操作系统

## ls -l

`ls -l` 可以列出目录下的文件和目录信息，比如

```
root@ikutarian:/etc# ls -l
total 788
drwxr-xr-x 3 root root    4096 Nov  6 10:26 acpi
-rw-r--r-- 1 root root    3028 Jul 31 08:30 adduser.conf
drwxr-xr-x 2 root root    4096 Nov  6 10:33 alternatives
drwxr-xr-x 3 root root    4096 Nov  6 10:26 apm
drwxr-xr-x 3 root root    4096 Nov  6 10:34 apparmor
drwxr-xr-x 9 root root    4096 Nov  6 10:26 apparmor.d
drwxr-xr-x 3 root root    4096 Nov  6 10:26 apport
drwxr-xr-x 6 root root    4096 Nov  6 11:39 apt
-rw-r----- 1 root daemon   144 Jan 15  2016 at.deny
```

这些字段都有自己的含义，以第一行 `drwxr-xr-x 3 root root    4096 Nov  6 10:26 acpi` 为例从左到右进行说明

`drwxr-xr-x` 是由 3 个部分组成的： `d` + `rwx` + `r-x` + `r-x`。`d` 表示这是一个目录，如果是文件就以 `-` 表示。后三个字符（`rwx`）是文件所有者的访问权限。再后面三个字符（`r-x`）是文件所属用户组的访问权限，最后三个字符（`r-x`）是所有人的访问权限。

`rwx` 中 `r` 是 read，表示可读；`w` 是 write，表示可写；`x` excute，表示可执行

`3` 表示硬链接的数目

`root` 表示文件所属用户

`root` 表示文件所属用户组

`4096` 表示文件的大小，以字节为单位

`Nov  6 10:26` 是文件最后修改日期

`acpi` 是目录名，如果是文件就是文件名

## 查看文件类型

Linux 不要求文件名要有后缀。如果先知道文件的类型，可以使用

```
file filename
```

来查看文件类型

# 5 操作文件和目录

## 通配符

可以用通配符来表示一些特定格式的文件名和目录名，所以通配符的知识也需要掌握

|通配符|意义|
|:--|:--|
|*|匹配任意多个字符（>= 0）|
|？|匹配任意一个字符（== 0）|
|[characters]|匹配任意一个属于字符集中的字符|
|[!characters]|匹配任意一个不在字符集中的字符|
|[[:class:]]|匹配任意一个指定字符类中的字符|

假设我要查找文件，用通配符来指代文件名

|模式|匹配对象|
|:--|:--|
|*|所有文件|
|g*|所有以字母“g”开头的文件|
|b*.txt|以“b”开头，中间又零个或多个字符，以“.txt”结尾的文件名|
|Data???|“Data”开头，后面跟着三个字符的文件名|
|[abc]*|以“a”或“b”或“c”开头的文件名|

一个新知识：字符类

|字符类|意义|
|:--|:--|
|[:alnum:]|匹配任意一个字母或数字|
|[:alpha:]|匹配任意一个字母|
|[:digit:]|匹配任意一个数字|
|[:lower:]|匹配任意一个小写字母|
|[:upper:]|匹配任意一个大写字母|

对于 `[a-z]` 和 `[A-Z]`，不要再使用它了，应该改用 `[:lower:]` 和 `[:upper:]`

## 创建目录

```
mkdir directory ...
```

## 复制文件和目录到指定的目录

复制一个文件或目录到指定的目录

```
cp item target_dir
```

复制多个文件或目录到指定目录

```
cp item ... target_dir
```

## 移动和重命名

```
mv item1 item2
```

## 删除

```
rm item
```

强制删除

```
rm -f item
```

删除目录

```
rm -r item
```

**重要**

> 删除文件之前，先执行 `ls` 命令确认要删除的内容

## 符号链接和硬链接

硬链接没什么用，学习符号连接才对。符号连接也叫软链接

```
ln -s item link
```

`-s` 就是表示软连接的 `soft` 的缩写

关于硬链接和符号链接，可以看这篇《{% post_link 理解Linux的硬链接和软链接 %}》

# 6 使用命令

## 命令有哪些类型？

可以使用 `type` 命令查看命令的类型

1. 可执行程序

比如位于 `r/bin/` 和 `/usr/bin/` 下的文件，比如 `cp`、`python`、`gcc` 等

使用 `type` 命令输出

```
root@iZ94fg0bhwgrgtZ:~# type python
python is /usr/bin/python
```

可以看到 `python is /usr/bin/python`

2. 内置的 shell 命令

比如 `cd`

```
root@iZ94fg0bhwgrgtZ:~# type cd
cd is a shell builtin
```

`type` 自己本身也是内置的 shell 命令

```
root@iZ94fg0bhwgrgtZ:~# type type
type is a shell builtin
```

3. 一个 shell 函数

一个加入到环境变量的 shell 脚本

4. 一个命令别名

`ls` 其实是一个命令别名

```
root@iZ94fg0bhwgrgtZ:~# type ls
ls is aliased to `ls --color=auto'
```

## which －显示可执行程序的路径

```
root@iZ94fg0bhwgrgtZ:~# which ls
/bin/ls
root@iZ94fg0bhwgrgtZ:~# which python
/usr/bin/python
```

## 查看命令的文档

`whatis` - 一句话解释命令的用途
`man` - 查看手册

有些命令还可以带上 `--help` 参数查看文档

## 命令的分隔符

使用 `;` 作为命令的分隔符

## 查看所有已经定义的别名

不要带参数

```
alias
```

## 给命令设置别名

两个步骤：

1. 查找别名是否已经被使用 
2. 创建别名

给 `cd /usr; ls; cd` 创建一个别名，比如 test。首先确认一下 test 是否被使用

```
root@iZ94fg0bhwgrgtZ:~# type test
test is a shell builtin
```

可以看到 test 是一个内置的 shell 命令。于是换一个名字，比如 foo

```
root@iZ94fg0bhwgrgtZ:~# type foo
-bash: type: foo: not found
```

`-bash: type: foo: not found` 表示 foo 没有被使用。那就用这个了

alias 的**格式**如下，`name='string'` 没有空格

```
alias name='string'
```

于是

```
alias foo='cd /usr; ls; cd'
```

**注意：**

如果输入 `exit` 退出账户的话，刚刚创建的别名就失效了。可以加到环境变量中，让别名不失效

# 7 重定向

## 标准输入、标准输出、标准错误

- stdin
- stdout
- stderr

stdout 和 stderr 通常连接的是屏幕，stdin 连接的是键盘

## 所谓重定向

就是改变 stdin 的来源，改变 stdout、stderr 的输出位置

## 重定向 stdout 到文件

使用 `>`

`>` 会覆盖或者新建文件内容。如果要向文件追加内容，可以使用 `>>`

## 重定向 stderr 到文件

使用 `2>`。比如，对一个不存在的目录使用 `ls`

```
ls foo 2> ls-out.txt
```

## 重定向 stdout、stderr 到同一个文件

使用 `&>`

列出不存在的目录 foo 和 ~ 目录的内容，并重定向到文件中

```
ls foo ~ &> ls-out.txt
```

## 重定向 stdin

使用 `<`，把输入源从键盘改成文件

```
cat < ls-out.txt
```

## 管道

`|`，前一个命令的输出作为下一个命令的输入

## 打印匹配文本

`grep`，可以打印匹配文本所在那一行的所有文本，可以使用正则表达式

## 把数据同时输出到 stdout 和文件

这条命令无法得到结果。因为 `|` 和 `>` 没办法同时用

```
ls -l ~ | > ls-out.txt
```

和管道连用时，可以用 `tee` 代替 `>`

```
ls -l ~ | tee ls-out.txt
```

# 9 键盘操作高级技巧

## 清空屏幕

```
clear
```

## 自动补全

键盘 Tab 键

## 命令历史记录

```
history
```

搜索历史记录，比如

```
history | grep /usr/bin
```

# 10 权限

**这是重点**

## 拥有者、组成员、其他人

这个文件或者文件夹是我的，我就是拥有者（Owener），我对它拥有控制权

而我又是属于一个用户组，我可以规定我的组员们对这个文件或者文件夹拥有怎么样的控制权

而对于组员之外的其他用户，我可以规定他们对这文件或者文件夹拥有怎么样的控制权

## 权限

- 读（read）
- 写（write）
- 执行（excute）

## 权限的表示法

- 八进制
- 符号

常用的就用“八进制”即可

|八进制|二进制|权限|
|:--:|:--:|:--:|
|0|000|---|
|1|001|--x|
|2|010|-w-|
|3|011|-wx|
|4|100|r--|
|5|101|r-x|
|6|110|rw-|
|7|111|rwx|

## chmod

权限，也叫 “File Mode”，所以 `chmod` 就是 “change file mode” 的缩写

```
root@iZ94fg0bhwgrgtZ:~# > foo.txt
root@iZ94fg0bhwgrgtZ:~# ls -l foo.txt 
-rw-r--r-- 1 root root 0 Nov 16 15:23 foo.txt
root@iZ94fg0bhwgrgtZ:~# chmod 600 foo.txt
root@iZ94fg0bhwgrgtZ:~# ls -l foo.txt 
-rw------- 1 root root 0 Nov 16 15:23 foo.txt
```

# 11 进程

## PID

系统会给每个进程分配一个数字，这个数字叫 PID（process id 的缩写）

## 查看进程快照

显示当前会话相关的进程快照

```
ps
```

以一定格式，输出所有进程快照的信息

```
ps aux
```

## 查看系统的动态信息

```
top
```

比如

```
top - 20:51:07 up 41 days,  6:59,  1 user,  load average: 0.06, 0.05, 0.02
Tasks:  90 total,   1 running,  89 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  1.0 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2048124 total,   110620 free,    96232 used,  1841272 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1742532 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
  
26673 root      20   0  805560  76512  38068 S  0.3  3.7  21:39.20 dockerd
```

下面逐行说明

```
top - 20:51:07 up 41 days,  6:59,  1 user,  load average: 0.06, 0.05, 0.02
```

|字段|意义|
|:--|:--|
|top|程序名|
|20:51:07|当前时间|
|up 41 days,  6:59|计算机从上次启动到现在的正常运行时间|
|1 user|有一个用户登陆系统|
|load average|加载平均值。分别表示最后60秒、前5分钟、前15分钟。若平均值低于1.0，表示计算机工作不忙碌|

```
Tasks:  90 total,   1 running,  89 sleeping,   0 stopped,   0 zombie
```

|字段|意义|
|:--|:--|
|Tasks|总结了进程数目和各种状态下进程的个数|

```
%Cpu(s):  0.3 us,  1.0 sy,  0.0 ni, 98.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

|字段|意义|
|:--|:--|
|%Cpu(s)|CPU正在执行的进程的特性|
|us|user process（用户进程）|
|sy|system process（内核进程）|
|ni|nice process，即低优先级进程|
|id|表示 98.7% 的 CPU 时间是空闲的|
|wa|0.0% 的 CPU 时间来等待 IO|


```
KiB Mem :  2048124 total,   110620 free,    96232 used,  1841272 buff/cache
```

物理内存的使用情况

```
KiB Swap:        0 total,        0 free,        0 used.  1742532 avail Mem 
```

交换分区（虚拟内存）的使用情况

## 给进程发送信号

```
kill [-signal] PID...
```

`[-signal]` 常用的是 `9`，表示让内核立刻杀死进程（进程来不及做收尾工作或者保存劳动成果）。比如 `kill -9 PID...`

# 12 环境变量

## 两个重要的配置文件

- `/etc/profile`
- `˜/.bashrc`

`/etc/profile` 是全局配置文件。`˜/.bashrc` 是用户自己的配置文件

以 Windows 为例

{% asset_img QQ截图20181120112009.png %}

`/etc/profile` 就相当于系统变量，`˜/.bashrc` 就相当于用户变量

## 注意

修改完配置文件之后，需要使用 `source` 命令才能生效

## PATH

就相当于 Windows 环境变量里的 Path

{% asset_img QQ截图20181120112009.png %}

可以用 `$PATH` 表示

```
echo $PATH
```

## 以安装 JDK 为例

下载 jdk-8u191-linux-x64.tar.gz 到 `/root/`，然后解压到 `/root/jdk1.8.0_191/` 文件夹中。打开 `etc/profile`，在最后面增加以下语句

```
JAVA_HOME=/root/jdk1.8.0_191
export JAVA_HOME
export PATH=$PATH:$JAVA_HOME/bin
```

让环境变量生效

```
source /etc/profile
```

现在测试一下 `echo $PATH`，可以看到末尾多出了 JDK 的路径

```
root@iZ94fg0bhwgrgtZ:~/jdk1.8.0_191# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/root/jdk1.8.0_191/bin
```

再测试一下 `java -version`，可以打出 JDK 的版本

```
root@iZ94fg0bhwgrgtZ:~/jdk1.8.0_191# java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```

# 13 vi 编辑器

## 模式

- 命令模式 - 键盘 `ESE`
- 插入模式 - 键盘 `i`

## 保存

- :w
- :wq

## 移动光标

- 上 - 键盘 `j`
- 下 - 键盘 `k`
- 左 - 键盘 `h`
- 右 - 键盘 `l`
- 行首 - 键盘 `0`
- 行尾 - 键盘 `$`
- 文件末尾 - 键盘 `G`

## 追加文字

- a - 当前位置的下一个位置进行追加
- A - 当前行的行尾进行追加

## 行插入

- o - 当前行的下方插入
- O - 当前行的上方插入

## 删除

- x - 删除当前字符
- dd - 删除当前行

## 复制粘贴

- yy - 复制当前行
- p - 粘贴当前行

## 查找

- / + 要查找的字符
- n - 下一个结果

# 15 软件包管理

## 更新软件源

Debian

```
apt-get update
```

RedHat

```
yum update
```

## 查找软件包

Debian

```
apt-cache search search_string
```

RedHat

```
yum search search_string
```

## 安装软件包

Debian

```
apt-get install package_name
```

RedHat

```
yum install package_name
```

## 卸载软件包

Debian

```
apt-get remove package_name
```

RedHat

```
yum erase package_name
```

# 19 归档和备份

## 压缩

```
gzip file_name
```

会生成一个 `file_name.gz` 文件名的文件，同时 `file_name` 文件消失了

## 解压缩

```
gzip -d file_name.gz
```

可以得到 `file_name` 文件名的文件，同时 `file_name.gz` 文件消失了

## 解压归档文件

比如 file_name.tar.gz 就是一个用 gzip 压缩过的 tar 归档文件，所以解压的时候，要使用

```
tar zxf file_name.tar.gz
```

`z` 表示这是一个用 gzip 压缩过的 tar 归档文件。`x` 表示把文件从 tar 包中提取出来。`f` 表示目标文件名，也就是 file_name.tar.gz

# 25 编写第一个 Shell 脚本

## 模板

文件名：hell_world.sh

```
#!/bin/bash
# This is our first script
echo 'Hello World'
```

## 权限

自己有 rwx 权限，其他人有 r-x 权限

```
chomod 755 hell_world.sh
```

注意：

> 为了能够执行脚本，脚本必须是可读的。所以 r-x 权限必不可少

## 运行

```
./hell_world.sh
```

或者也可以加入环境变量

# 命令总结

- 常用命令
  - `exit` - 退出终端
  - `shutdown now` - 关机
  - `date` - 查看当前日期
  - `cal` - 查看本月日历
  - `df -h` - 查看硬盘使用量
  - `free -h` - 查看内存使用量
- 文件系统
  - `pwd` - 查看当前的工作目录
  - `ls` - 列出目录内容
  - `cd` - 文件夹跳转
  - `file` - 查看文件类型
  - `mkdir` - 创建目录
  - `cp` - 复制文件或目录
  - `mv` - 移动和重命名
  - `rm` - 删除文件或目录
  - `ln` - 创建链接
- 使用命令
  - `type` - 命令查看命令的类型
  - `which` －显示可执行程序的路径
  - `whatis` - 一句话解释命令的用途
  - `man` - 查看手册
  - `alias` - 查看所有已经定义的别名
  - `alias name='string'` - 创建命令的别名
- 重定向
  - `>` - 重定向 stdout 到文件
  - `2>` - 重定向 stderr 到文件
  - `&>` - 重定向 stdout、stderr 到同一个文件
  - `<` - 重定向 stdin
  - `|` - 管道
  - `grep` - 打印匹配文本所在那一行的所有文本
  - `tee` - 把数据同时输出到 stdout 和文件
- 键盘操作高级技巧
  - `clear` - 清空屏幕
  - 键盘 Tab 键 - 自动补全
  - `history` - 命令历史记录
- 权限
  - `chmod` - 更改权限
  - `sudo`- 获取超级用户权限
- 进程
  - `ps` - 查看进程快照
  - `top` - 查看系统的动态信息
  - `kill` - 给进程发送信号
- 环境变量
  - `/etc/profile` - 全局配置文件
  - `~/.bashrc` - 用户自己的配置文件
  - `source` - 让配置文件生效
- 软件包管理
  - `apt-get update` - 更新软件源
  - `apt-cache search search_string` - 查找软件包
  - `apt-get install package_name` - 安装软件包
  - `apt-get remove package_name` - 卸载软件包
- 归档和备份
  - `gzip file_name` - 压缩
  - `gzip -d file_name` - 解压缩
  - `tar zxf file_name.tar.gz` - 加压缩归档文件