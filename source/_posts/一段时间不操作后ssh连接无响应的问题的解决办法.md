---
title: 一段时间不操作后ssh连接无响应的问题的解决办法
date: 2018-09-26 10:32:26
tags:
  - SSH
  - Linux
categories:
  - Linux
---

在 OSX 10.12.6 在控制台输入

```
ssh root@xxx.xxx.x.x.x
```

连接到 Ubuntu 16.04.4 LTS。发现了一个问题：如果一段时间不操作，控制台会出现无响应的问题。

<!-- more -->

经过查找资料，说是 Linux 到安全设置问题：如果 60s 内没有任何数据，将会自动断开 SSH 连接。所以需要修改一下安全设置的配置

本地，也就是 OSX 10.12.6

```
# 打开
sudo vim /etc/ssh/ssh_config
# 添加
ServerAliveInterval 50
ServerAliveCountMax 3 
```

远程，也就是 Ubuntu 16.04.4 LTS

```
# 打开
sudo vim /etc/ssh/sshd_config
# 添加
ClientAliveInterval 50
ClientAliveCountMax 3 
```

经过测试，一段时间不操作，SSH 也不会卡死了