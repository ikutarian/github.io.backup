---
title: git切换分支前需要做的操作
date: 2019-11-19 13:51:07
tags:
  - git
  - 分支
categories:
  - git
---

分支切换在 git 的使用中是一个很常见的情形。那么在切换分支之前需要做什么操作呢？

<!-- more -->

《Pro Git》中有一句话

> 切换分支的时候最好保持一个清洁的工作区域

所以，有如下几种处理方式：

## add、commit

在当前分支 A 执行 add、commit，然后切换到分支 B。在分支 B 做完工作之后，也进行 add、commit。两个分支互不影响，是最好的办法

## add、stash

add 但不 commit，接着 stash，把内容暂存，然后进行分支切换。当另外一个分支的工作做完之后，切换会当前分支，再执行 stash apply，继续工作