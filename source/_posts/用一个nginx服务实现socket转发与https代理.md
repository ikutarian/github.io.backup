---
title: 用一个nginx服务实现socket转发与https代理
date: 2019-03-27 16:09:24
tags:
  - Socket
  - 转发
  - Socket转发
  - Nginx
categories:
  - 安装与配置
---

之前我是用 Nginx 实现 Socket 转发，然后使用 Squid 实现 http/https 代理。现在可以用一个 Nginx 就实现这两个功能

<!-- more -->

# 环境

- CentOS release 6.4 (Final)
- nginx-1.14.2.tar.gz

# 编译安装 Nginx

要实现 http/https 代理需要额外的模块，所以得使用源码编译的方式来安装 Nginx

## 下载 Nginx 源码包

下载 Nginx 的源码包然后解压

```
wget http://nginx.org/download/nginx-1.14.2.tar.gz
tar -zxf nginx-1.14.2.tar.gz 
cd nginx-1.14.2
```

假设源码包的存放位置是 `/root/nginx/nginx-1.14.2`

## 安装PCRE、Zlib 和 OpenSSL

Nginx 依赖 PCRE、Zlib 和 OpenSSL，所以要安装一下

```
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```

## 下载 ngx_http_proxy_connect_module

ngx_http_proxy_connect_module 模块就是用来实现 http/https 代理的，具体可以看[官网](https://github.com/chobits/ngx_http_proxy_connect_module)

下载并解压

```
wget https://codeload.github.com/chobits/ngx_http_proxy_connect_module/zip/master
unzip master
```

假设存放位置是 `/root/nginx/ngx_http_proxy_connect_module-master`

## 安装

前面已经假设了：

- 假设 nginx 源码包的存放位置是 `/root/nginx/nginx-1.14.2`
- 假设 ngx_http_proxy_connect_module 存放位置是 `/root/nginx/ngx_http_proxy_connect_module-master`

首先进入 Nginx 源码包的目录

```
cd /root/nginx/nginx-1.14.2
```

然后执行 

```
patch -p1 < /root/nginx/ngx_http_proxy_connect_module-master/patch/proxy_connect_1014.patch
```

然后指定要编译的模块

```
./configure --add-module=/root/nginx/ngx_http_proxy_connect_module-master --with-http_ssl_module --with-stream
```

`--add-module=/root/nginx/ngx_http_proxy_connect_module-master` 这个就是 ngx_http_proxy_connect_module 模块的位置，`--with-http_ssl_module` 这个是 https 要用到的模块，`--with-stream` 这个是 Socket 转发需要的模块

执行完毕就能看到一个报告

```
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

这样说明模块指定没有问题，于是接着执行

```
make
make install
```

# 配置

进入 `/usr/local/nginx/conf`，新建一个文件 `proxy.conf`，填入如下内容

```
worker_processes  1;

events {
    worker_connections  1024;
}

# Socket转发 ---开始----
stream {
    upstream trans {
        # 要转发的目标服务器的 IP 与端口
        server 110.76.15.107:443;
    }

    server {
        # 本机监听端口
        listen 8008;
        proxy_pass trans;
    }
}
# Socket转发 ---结束--

http {

    # http/https代理 ---开始----
    server {
        listen                         8009;

        # dns resolver used by forward proxying
        resolver                       223.5.5.5;

        # forward proxy for CONNECT request
        proxy_connect;
        proxy_connect_allow            443 563;
        proxy_connect_connect_timeout  10s;
        proxy_connect_read_timeout     10s;
        proxy_connect_send_timeout     10s;

        # forward proxy for non-CONNECT request
        location / {
            proxy_pass http://$host;
            proxy_set_header Host $host;
        }
    }
    # http/https代理 ---结束----
}
```

# 启动

## 检查配置文件

首先检测配置文件的正确性

```
/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/proxy.conf
```

如果输出

```
nginx: the configuration file /usr/local/nginx/conf/proxy.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/proxy.conf test is successful
```

说明没问题

## 启动

```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/proxy.conf
```

使用如下命令可以查看 Nginx 目前监听的端口

```
netstat -anp | grep nginx
```

# Nginx 如何平滑添加 stream 模块

我在安装完 ngx_http_proxy_connect_module 模块之后才想起来，我忘了安装 stream 模块了。这个也有补救办法

进入 Nginx 的源码目录

```
cd /root/nginx/nginx-1.14.2
```

重新指定一下编译配置

```
./configure --add-module=/root/nginx/ngx_http_proxy_connect_module-master --with-http_ssl_module --with-stream
```

然后执行

```
make
```

**注意**

> 此处一定不能使用 `make install` 命令，执行该命令会将原有 Nginx 目录进行覆盖

关闭 nginx 服务

```
/usr/local/nginx/sbin/nginx -s quit
```

备份原有 Nginx 二进制文件

```
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx-no-strem
```

复制新编译好的 Nginx 二进制文件

```
cp ./objs/nginx /usr/local/nginx/sbin/nginx
```

然后再执行一下 `/usr/local/nginx/sbin/nginx -V` 就可以看到模块信息了

```
nginx version: nginx/1.14.2
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --add-module=/root/nginx/ngx_http_proxy_connect_module-master --with-http_ssl_module --with-stream
```