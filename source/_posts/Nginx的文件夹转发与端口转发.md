---
title: Nginx的文件夹转发与端口转发
date: 2018-10-09 23:09:33
tags:
  - Nginx
  - 文件夹转发
  - 端口转发
categories:
  - 安装与配置
---

我觉得 nginx 的强大与方便之处，就在于转发功能。nginx 可以提供两种转发：

- 文件夹转发
- 端口转发

文件夹转发常用于静态资源文件，比如访问 `img.ikutarian.com/filename.jpg` 就可以获取服务器硬盘上名为 `filename.jpg` 的文件

端口转发常用于反向代理。比如访问 `www.ikutarian.com` 可以把请求转发到 `localhost:8080` 的 Tomcat 上

<!-- more -->

现在就来实验以下这两种转发。因为我没有域名，而且是在本机上测试，所以要改一下 hosts 文件，把 `img.ikutarian.com` 和 `www.ikutarian.com` 指向到 `127.0.0.1`。打开 C:\Windows\System32\drivers\etc\hosts 文件，加入两条

```
127.0.0.1 img.ikutarian.com
127.0.0.1 www.ikutarian.com
```

## 环境

- Windows 10
- nginx-1.14.0

## 安装 nginx

打开[官网](http://nginx.org/en/download.html)，选择“Stable version”的版本下载，我这里选择的是 nginx-1.14.0。解压到某一个目录，比如 `E:\nginx-1.14.0`

## 运行、重启与关闭

打开 CMD，进入 `E:\nginx-1.14.0` 目录

```
start nginx
```

即可启动。打开 http://localhost 即可看到网页了，如下图

{% asset_img 20181010090305.png %}

对于重启、关闭等操作，nginx 是通过命令行

```
nginx.exe -s single
```

来实现的。常用 single 有以下几种

- stop — 快速关闭 nginx
- quit — 优雅地关闭 nginx
- reload — 重新加载配置文件

所以，对于重启，只需要输入

```
nginx.exe -s reload
```

对于关闭，通常使用

```
nginx.exe -s quit
```

## 一个有用的命令

当编辑完配置文件之后，想检查以下配置文件有没有没写对，nginx 提供了一个命令帮助我们检查

```
nginx.exe -t
```

`-t` 就是 test 的意思

## 转发配置前置工作

在 conf 文件夹下再创建一个 vhost 文件夹

打开 `conf\nginx.conf`，在尾部加入一条 `include vhost/*.conf;`。目的是把配置文件隔离开来，不让配置文件混在一起

```
...

http {

    ...

    include vhost/*.conf;
}
```

### 文件夹转发

在 vhost 文件夹下创建一个 `img.ikutarian.com.conf` 文件，注意一下这个文件名，这样写比较容易维护

加入以下配置信息

```
server {
    listen 80;
    server_name img.ikutarian.com;

    location / {
        root F:\IDEA\mmall\img;
    }
}
```

这个配置文件表示，监听 80 端口，host 为  img.ikutarian.com 的 http 请求，把请求转发到 `F:\IDEA\mmall\img` 文件夹下

**注意**：文件夹名称后面不要加 `\`，在 Windows 平台下的 nginx 是不允许的

然后就是检测配置文件，并重启 nginx

```
nginx.exe -t
nginx.exe -s reload
```

现在，我要访问 `F:\IDEA\mmall\img` 文件夹下一张名为 `img.jpg` 的图片，可以在浏览器输入 http://img.ikutarian.com/img.jpg 即可访问到

### 端口转发

类似的，在 vhost 文件夹下创建一个 `www.ikutarian.com.conf` 文件，文件名这么写比较好维护，这一点再次需要注意

加入配置

```
server {
    listen 80;
    server_name www.ikutarian.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

接着检测配置文件，重启 nginx

```
nginx.exe -t
nginx.exe -s reload
```

现在访问 http://www.ikutarian.com 就可访问到 localhost:8080 了

## 总结

- 文件夹转发，使用 `root`
- 端口转发，使用 `proxy_pass`

## 额外的内容

下面这张图应该不陌生，经常可以在网上看到

{% asset_img 20181010101903.png %}

这个其实也是利用了 nginx 的文件夹转发

在 vhost 下加入一个配置文件 file.ikutarian.com。别忘了把 file.ikutarian.com 加入 hosts，这样才能在本地测试

```
server {
    listen 80;
    server_name file.ikutarian.com;
    
    location / {
        # 打开目录浏览功能
        autoindex on;
        # 显示出文件的大概大小，单位是kB或者MB或者GB
        autoindex_exact_size off;
        # 显示的文件时间为文件的服务器时间
        autoindex_localtime on;

        root G:\smart-framework;
    }
}
```

**注意**：root 的路径不能带中文，而且末尾没有 `\`

这样访问 http://file.ikutarian.com/ 就可以看到效果了

{% asset_img 20181010102330.png %}
