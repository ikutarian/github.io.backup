---
title: Linux下SpringBoot应用的管理脚本
date: 2020-02-22 22:09:16
tags:
  - Shell
  - 脚本
categories:
  - 安装与配置
---

通常我们会使用 `jar -jar xxx.jar` 启动 SpringBoot 应用，然后 CTRL + C 或者 kill 进程关闭 SpringBoot。但是可不可以使用脚本来是帮我们做这些工作呢？

<!-- more -->

# 启动

一种是静默启动

```shell
#!/usr/bin/env bash

nohup java -jar demo-0.0.1.jar >/dev/null 2>&1 &
```

一种会打印消息到 `nohup.out` 文件中

```sh
#!/usr/bin/env bash

nohup java -jar demo-0.0.1.jar &
```

将上面的脚本保存为 `start.sh`，使用以下命令启动

```
bash start.sh
```

或者先授权

```
chmod +x start.sh
```

执行脚本

```
./start.sh
```

# 停止

```shell
#!/usr/bin/env bash

# 得到jar包运行的进程ID
pid=`ps -ef | grep demo-0.0.1.jar | grep -v grep | awk '{print $2}'`

# 如果找不到进程ID
if [ -z "${pid}" ]
then
    # 提示进程已停止
    echo application is already stopped
# 如果进程ID存在
else
    # kill进程
    echo kill ${pid}
    kill -9 ${pid}
fi
```

将上面的脚本保存为 `stop.sh`，使用以下命令启动

```
bash stop.sh
```

或者先授权

```
chmod +x stop.sh
```

执行脚本

```
./stop.sh
```

# 重启

```shell
#!/usr/bin/env bash

echo stop application
bash start.sh

echo start appliction
bash stop.sh
```

或者

```shell
#!/usr/bin/env bash

echo stop application
./start.sh

echo start appliction
./stop.sh
```