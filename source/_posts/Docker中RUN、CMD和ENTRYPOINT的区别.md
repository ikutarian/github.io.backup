---
title: Docker中RUN、CMD和ENTRYPOINT的区别
date: 2019-03-06 10:13:17
tags:
  - Docker
  - Dockerfile
  - RUN
  - CMD
  - ENTRYPOINT
categories:
  - Docker
---

Docker 中 RUN、CMD 和 ENTRYPOINT 的区别算是一个比较难理解的知识点，总结一下

<!-- more -->

# 作用

RUN

> 执行命令并创建新的 image layer

CMD

> 1. 容器启动后默认执行的命令
> 2. 如果使用 `docker run` 命令启动容器时指定了其他命令，CMD 指令将被忽略

ENTRYPOINT

> 1. 配置容器启动时运行的命令
> 2. 不会被忽略，一定会执行

# 命令的格式

RUN、CMD 和 ENTRYPOINT 都支持两种格式的命令：Shell 和 Exec

## Shell 格式

```
<instruction> <command>
```

比如

```dockerfile
RUN apt-get install python3
CMD echo "Hello world"
ENTRYPOINT echo "Hello world"
```

Shell 格式的底层实现是调用 `/bin/sh -c [command]` 来实现的

## Exec 格式：

```
<instruction> ["executable", "param1", "param2", ...]
```

例如

```dockerfile
RUN ["apt-get", "install", "python3"]
CMD ["/bin/echo", "Hello world]
ENTRYPOINT ["/bin/echo", "Hello world"]
```

Exec 格式底层实现是把命令用空格分隔拼接起来，得到一个新的命令 `[command]`，然后直接调用 `[command]`

## 当使用环境变量时

比较一下 `CMD echo $PATH` 和 `CMD ["echo", "$PATH"]` 的区别

执行 `CMD echo $PATH` 时，输出

```
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

使用 `docker ps -l --no-trunc` 观察 `COMMAND` 部分的内容，可以看到 `CMD echo $PATH` 变成了 `/bin/sh -c 'echo $PATH'`

```
[node1] (local) root@192.168.0.13 ~/env
$ docker ps -l --no-trunc
CONTAINER ID                                                       IMAGE                 COMMAND CREATED             STATUS                      PORTS               NAMES
e12085bd0f2c0aff932da620d70d82a42b4c097e2dd8ae51b5a6d1f396d5ea90   ikutarian/env-shell   "/bin/sh -c 'echo $PATH'" 18 seconds ago      Exited (0) 17 seconds ago                       modest_swanson
```

执行 `CMD ["echo", "$PATH"]` 时，输出

```
$PATH
```

使用 `docker ps -l --no-trunc` 观察 `COMMAND` 部分的内容，可以看到 `CMD ["echo", "$PATH"]` 还是保持原样

```
$ docker ps -l --no-trunc
CONTAINER ID                                                       IMAGE                COMMAND             CREATED            STATUS                     PORTS               NAMES
e448a81c9bed9b8e9a6b5bc4317fcad97ef3c795e139dcdfdac6419e8d3a3ab3   ikutarian/env-exec   "echo $PATH"        4 seconds ago       Exited (0) 2 seconds ago                       musing_ritchie
```

所以得出结论：

1. Shell 格式的命令，会被包装为 `sh -c` 的参数的形式进行执行
2. Exec 格式的命令则是原样执行。因此如果需要使用环境变量的话，应该写成这样：`CMD ["/bin/bash", "-c", "echo $PATH"]`

## 建议

1. RUN 指令使用 Shell 格式的命令即可
2. CMD 和 ENTRYPOINT 指令推荐使用 Exec 格式的命令

# ENTRYPOINT 和 CMD 组合使用

记住几点：

1. CMD 设置容器启动时执行的命令，可以被 `docker run` 覆盖
2. ENTRYPOINT 也是设置容器启动时执行的命令，无法被 `docker run` 覆盖，一定会执行

所以，可以将 ENTRYPOINT 和 CMD 组合使用：

> ENTRYPOINT 指定默认的运行命令，CMD指定默认的运行参数

比如这个 Dockerfile

```dockerfile
FROM busybox
ENTRYPOINT ["/bin/ping", "-c", "3"]
CMD ["localhost"]
```

构建镜像，然后执行

```
docker build -t ikutarian/ping .
docker run ikutarian/ping
```

输出

```
PING localhost (127.0.0.1): 56 data bytes
64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.254 ms
64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.069 ms
64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.048 ms

--- localhost ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.048/0.123/0.25
```

使用 `docker ps -a --no-trunc`，查看一下 `COMMAND` 部分的内容

```
$ docker ps -l --no-trunc
CONTAINER ID  IMAGE            COMMAND                      CREATED             STATUS                         PORTS  NAMES
41b4661a0     ikutarian/ping   "/bin/ping -c 3 localhost"   2 minutes ago       Exited (0) About a minute ago         zen_ptolemy
```

可以看到容器启动时，执行的命令是 `/bin/ping -c 3 localhost`

上面的命令是用 `ENTRYPOINT ["/bin/ping", "-c", "3"]` 和 `CMD ["localhost"]` 组成而成的。CMD 指令的内容可以被 `docker run` 覆盖，如果希望容器启动之后 ping 的不是 localhost 而是其他的，可以这么做

```
docker run ikutarian/ping www.google.com
```

现在的输出结果是

```
PING www.google.com (172.217.7.196): 56 data bytes
64 bytes from 172.217.7.196: seq=0 ttl=49 time=1.057 ms
64 bytes from 172.217.7.196: seq=1 ttl=49 time=9.893 ms
64 bytes from 172.217.7.196: seq=2 ttl=49 time=1.232 ms

--- www.google.com ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.057/4.060/9.893 ms
```

用 `docker ps -l --no-trunc` 查看一下 `COMMAND` 的信息

```
$ docker ps -l --no-trunc
CONTAINER ID   IMAGE            COMMAND                           CREATED              STATUS                          PORTS   NAMES
d01d1ebd4      ikutarian/ping   "/bin/ping -c 3 www.google.com"   About a minute ago   Exited (0) About a minute ago           quirky_hofstadter
```

## 注意点

1. Shell 格式的底层实现是调用 `/bin/sh -c [command]` 来实现的
2. Exec 格式底层实现是把命令用空格分隔拼接起来，得到一个新的命令 `[command]`，然后直接调用 `[command]`

所以 ENTRYPOINT 和 CMD 组合使用时，应该使用 Exec 格式的命令，否则的话就得不到正确的结果

```dockerfile
ENTRYPOINT /bin/ping -c 3
CMD localhost               
# 得到 /bin/sh -c '/bin/ping -c 3' /bin/sh -c localhost


ENTRYPOINT ["/bin/ping","-c","3"]
CMD localhost               
# 得到 /bin/ping -c 3 /bin/sh -c localhost


ENTRYPOINT /bin/ping -c 3
CMD ["localhost"]      
# 得到 /bin/sh -c '/bin/ping -c 3' localhost


ENTRYPOINT ["/bin/ping","-c","3"]
CMD ["localhost"]           
# 得到 /bin/ping -c 3 localhost
```

从上面看出, 只有 ENTRYPOINT 和 CMD 都用 Exec 格式的命令, 才能得到预期的效果

# 结论

如果你想让你的 docker image 做真正的工作, 一定会在 Dockerfile 里用到 ENTRYPOINT 或是 CMD。这2个命令不是互斥的. 在很多情况下, 可以组合 ENTRYPOINT 和 CMD 命令, 提升最终用户的体验