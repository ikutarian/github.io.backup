---
title: /bin/sh -c 是什么意思?
date: 2019-02-25 15:55:42
tags:
  - 命令行
  - bash
categories:
  - Linux
---

编写 Dockerfile 时，经常使用到 `/bin/sh -c` 命令，一直不懂是什么意思

<!-- more -->

执行

```
man /bin/sh
```

打开 man page，查看说明文档，可以看到相关解释

```
 -c    Read commands from the command_string operand instead of from the standard input. Special parameter 0 will be set from the command_name operand and the positional parameters ($1, $2, etc.)  set from the remaining argument operands.
```

也就是说，如果带上 `-c` 参数的话，会把参数后面的值当作命令行，而不是普通的字符串。比如执行如下命令

```
/bin/sh ls
```

会输出

```
root@iZ94fg0bhwgrgtZ:/# /bin/sh ls
/bin/sh: 0: Can't open ls
```

如果带上了 `-c` 参数，变成这样 `/bin/sh -c ls`，会输出

```
root@iZ94fg0bhwgrgtZ:/# /bin/sh -c ls
bin   dev  home        initrd.img.old  lib64	   media  opt	root  sbin  sys  usr  vmlinuz
boot  etc  initrd.img  lib	       lost+found  mnt	  proc	run   srv   tmp  var  vmlinuz.old
```

可以看到输出结果和执行 `ls` 命令的结果是一样的。`/bin/sh` 带上了 `-c` 参数就会把后面的字符串当作命令执行，在这里就是执行了 `ls` 命令

如果 `-c` 后面的命令行有空格隔开，可以使用双引号括起来，比如

```
/bin/sh -c "ls -al"
```

这样就相当于执行了 `ls -al` 命令