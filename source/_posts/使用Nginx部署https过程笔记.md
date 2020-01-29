---
title: 使用Nginx部署https过程笔记
date: 2019-12-13 11:17:47
tags:  
  - nginx
  - https
categories:
  - nginx
---

本文说明了在 Nginx 服务器中安装 SSL 证书的过程与经验

<!-- more -->

# 说明

- 域名以 `www.domain.com` 为例
- Nginx 版本以 `1.15.7` 为例
- 操作系统为 `Windows Server 2008 R2 Enterprise` 为例

# 前提条件

- 已买好了 https 证书
- 已安装好 Nginx 

# 证书安装

在 Nginx 的 `conf` 文件夹下新建一个 `cert` 文件夹，将已获取的 `www.domain.com.pem` 证书文件和 `www.domain.com.key` 私钥文件拷贝到 `cert` 文件夹中

# 创建配置文件的目录

在 `conf` 下新建一个文件夹 `vhost`，用来放置 `server` 的配置

编辑 `conf/nginx.conf` 文件，在 `http` 代码块的最后一行添加

```nginx
include vhost/*.conf;
```

# 配置 https

在 `conf/vhost` 下新建一个文件 `www.domain.com.conf`，添加如下配置

```nginx
server {
  # 监听https的443端口
	listen 443 ssl;
  # 证书绑定的域名
  server_name www.domain.com;
	
  # 证书文件的位置
  ssl_certificate cert/www.domain.com.pem;
  # 私钥文件的位置
  ssl_certificate_key cert/www.domain.com.key;
  ssl_session_timeout 5m;
  # 配置加密套件
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  # 配置协议
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
	
  # 静态页面
  location / {
  	root D:\\www.domain.com;
    index index.html
  }

  # 后端API
  location /api {
  	proxy_pass http://127.0.0.1:4396;
  }	
}

server {
  # 监听80端口
  listen 80;
  # 证书绑定的域名
  server_name www.domain.com;
  # http重定向为https
  rewrite ^/(.*) https://$server_name/$1 permanent;
}
```

可以注意到，这个配置文件配置了两个 `server` 代码块。这是因为一个监听 443 端口，一个监听 80 端口。

当访问 `http://www.domain.com` 时，将进入监听 80 端口的 `server` 代码块，它会将 http 重定向为 https，进入 监听 443 端口的 `server` 代码块

如果访问的是 `https://www.domain.com`，将会进入监听 443 端口的 `server` 代码块

## 特殊情况

当使用同一个 IP 地址去配置两个或更多的 HTTPS 服务器的时候，会出现证书不匹配的情況

比如两个域名 `www.example.com` 和 `www.example.org`，它们的 IP 都是一样的，Nginx 的配置如下

```nginx
server {
  listen          443 ssl;
  server_name     www.example.com;
  ssl_certificate www.example.com.crt;
  #...
}

server {
  listen          443 ssl;
  server_name     www.example.org;
  ssl_certificate www.example.org.crt;
  #...
}
```

如果浏览器请求 `www.example.org`，得到的证书会是默认的 `www.example.com.crt`，而不是 `www.example.org.crt`。这是因为 SSL 协议行为所致，SSL 连接在浏览器发送 HTTP 请求之前就被建立，Nginx 并不知道被请求的服务器名字，因此 Nginx 只会提供默认的服务器证书

解決这个问题最原始最有效的方法就是为每个 HTTPS 服务器分配独立的 IP 地址：

```nginx
server {
    listen          192.168.1.1:443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    #...
}

server {
    listen          192.168.1.2:443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    #...
}
```

# 背景知识

## 证书的分类

SSL 证书一共有 4 种

- 企业级别：EV(Extended Validation)、OV(Organization Validation)
- 个人级别：IV(Identity Validation)、DV（Domain Validation）

EV、OV、IV 是收费的，DV 是免费的

## 应用 HTTPS 后的好处

1. 防流量劫持

全站 HTTPS 是根治运营商、中间人流量劫持的解决方案，不仅可以杜绝网页中显示的小广告，更可以保护用户隐私安全

2. 提升搜索排名

采用 HTTPS 可以帮忙搜索排名的提升，提高站点的可信度和品牌形象

3. 杜绝钓鱼网站

HTTPS 地址栏绿色图标可以帮助用户识别出钓鱼网站，保障用户和企业的利益不受损害，增强用户信任

# 参考

- [Configuring HTTPS servers](https://nginx.org/en/docs/http/configuring_https_servers.html)
- [Nginx 服务器证书安装](https://cloud.tencent.com/document/product/400/35244)
- [Nginx 配置 HTTPS 服务器](https://aotu.io/notes/2016/08/16/nginx-https/index.html)