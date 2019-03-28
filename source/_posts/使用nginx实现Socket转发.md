---
title: 使用nginx实现Socket转发
date: 2019-03-27 11:09:32
tags:
  - Socket
  - 转发
  - Socket转发
  - Nginx
categories:
  - 安装与配置
---

Nginx 常见的用途是反向代理，其实它也能实现 Socket 转发

<!-- more -->

# 前提

要实现 Socket 转发，需要 `stream` 模块，可以使用 `nginx -V` 命令查看一下目前已经有的模块

```
[root@asdfsd ~]# nginx -V
nginx version: nginx/1.14.2
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-23) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

可以看到 `--with-stream` 是有的，这样就可以实现 Socket 转发了

# 配置

打开 `/etc/nginx/nginx.conf`，加入下面这段配置

```
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
# Socket转发 ---结束----
```

加入配置之后的完整配置文件如下

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


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
# Socket转发 ---结束----

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

# 启动

先检测配置文件是否正确无误

```
nginx -t
```

然后启动

```
nginx
```

如果要重启的话

```
nginx -s reload
```

然后让别人连上我的 `ip:端口`，我就能把 Socket 转发到配置好的目标服务器了