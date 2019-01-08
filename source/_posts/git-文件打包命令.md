---
title: git 文件打包命令
date: 2018-09-17 10:39:41
tags:
  - Git
categories:
  - Git
---

使用的命令 `git archive`

文档在 [http://git-scm.com/docs/git-archive](http://git-scm.com/docs/git-archive) 

简单的用法就是

```
git archive --format zip --output /path/to/file.zip master # 将 master 以zip格式打包到指定文件
```

还有个更简单的

```
git archive v0.1 | gzip > site.tgz
git archive master > /home/hainuo/fds.zip
```

