---
title: 理解Linux的硬链接和软链接
date: 2018-11-13 11:18:09
tags:
  - Linux
  - 硬链接
  - 软链接
categories:
  - Linux
---

硬链接和软连接理解起来其实就那么回事，底层原理知道了就好理解了

<!-- more -->

## 底层原理

一个文件包含文件名和数据。文件在 Linux 被分成两个部分：用户数据（user data）和元数据（metadata）。

用户数据，即文件数据块（data block），数据块是记录文件真实内容的地方。而元数据则是文件的附加属性，比如文件大小、创建时间、所有者信息等

对用户来说，文件名是文件的唯一标识，但是在 Linux 内部来说，元数据中的 inode 号才是文件的唯一标识。文件名是为了方便人们记忆和使用，Linux 是通过 inode 号来找到文件数据块的

{% asset_img image001.jpg %}

可以使用 `stat` 来查看一个文件的详细信息

一个包含一行文本的文件 myfile

```
echo "This is a plain text file." > myfile
```

调用 `stat` 命令

```
stat myfile
```

输出

```
root@iZ94fg0bhwgrgtZ:~# stat myfile 
  File: 'myfile'
  Size: 27        	Blocks: 8          IO Block: 4096   regular file
Device: fd01h/64769d	Inode: 1185274     Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2018-11-13 11:28:24.335830539 +0800
Modify: 2018-11-13 11:28:21.071826491 +0800
Change: 2018-11-13 11:28:21.071826491 +0800
 Birth: -
```

可以看到 inode 号是 `1185274`

也可以使用 `ls -li` 看到目录下的文件和目录的 inode 号信息

```
root@iZ94fg0bhwgrgtZ:~# ls -li
total 76
1188629 drwxr-xr-x 2 root root  4096 Nov  6 17:28 docker_img
1184255 drwxr-xr-x 4 root root  4096 Nov 10 16:37 download
1185295 -rw-r--r-- 1 root root   243 Oct  8 13:50 dump.rdb
1185274 -rw-r--r-- 1 root root    27 Nov 13 11:28 myfile
1185493 -rw-r--r-- 1 root root 58355 Sep 25 10:59 redis.conf
```

最前面的这些数字就是 inode 号

```
1188629
1184255
1185295
1185274
1185493
```

## 实验

创建一个文件，写入一些数据

```
touch myfile && echo "This is a plain file" > myfile
```

创建一个硬链接

```
ln myfile hard
```

查看 myfile 和 hard 的 inode 号。可以看到文件 myfile 和硬链接 hard 的 inode 都是 `1185276`，文件属性都是 `-`，表示他们都是一个文件

```
root@iZ94fg0bhwgrgtZ:~/test# ls -li
total 8
1185276 -rw-r--r-- 2 root root 21 Nov 13 11:49 hard
1185276 -rw-r--r-- 2 root root 21 Nov 13 11:49 myfile
```

再创建一个软链接

```
ln -s myfile soft
```

调用 `li -li`。发现 inode 不一样了，而且文件名表示方式也不一样，变成 `soft -> myfile`，soft 的文件属性是 `l`，表示它是一个链接

```
root@iZ94fg0bhwgrgtZ:~/test# ls -li
total 8
1185276 -rw-r--r-- 2 root root 21 Nov 13 11:49 hard
1185276 -rw-r--r-- 2 root root 21 Nov 13 11:49 myfile
1195389 lrwxrwxrwx 1 root root  6 Nov 13 14:02 soft -> myfile
```

现在把 myfile 删除，然后分别输出 hard 和 soft 的内容

```
root@iZ94fg0bhwgrgtZ:~/test# rm myfile 
root@iZ94fg0bhwgrgtZ:~/test# cat hard 
This is a plain file
root@iZ94fg0bhwgrgtZ:~/test# cat soft 
cat: soft: No such file or directory
```

删掉了 myfile 之后，仍然可以输出文件内容。因为 hard 的 inode 还在，仍然指向数据块，所以可以查看到文件内容。而 soft 的 inode 指向的只是一个文件路径。现在这个文件路径指向的文件被删除了，自然没办法查看到文件内容了

{% asset_img image002.jpg %}

## 总结

- 硬链接：和文件的 inode 号一样。就**好像一个文件有多个文件名**
- 软链接：和文件的 inode 号不一样。**存放的是文件路径，就像 Windows 下的快捷方式**
- 推荐使用软链接

## 参考

- [理解 Linux 的硬链接与软链接](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/index.html)
- [5分钟让你明白“软链接”和“硬链接”的区别](https://www.jianshu.com/p/dde6a01c4094)