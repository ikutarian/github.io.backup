---
title: Nginx中location与proxy_pass的匹配规则
date: 2019-07-10 10:51:22
tags:  
  - Nginx
  - 前后端分离
  - location
  - proxy_pass
  - 路径匹配
categories:
  - Nginx
---

`location` 与 `proxy_pass` 的匹配规则，在 Nginx 中算是一个难点。而且在配置前后端分离项目的时候也很重要。因此总结一下

<!-- more -->

现在来做一个实验。使用 npm 安装一个 http-server

```
npm install http-server -g
```

它是一个静态服务器，对于客户端的每一个请求都会在控制台打印出信息，可以利用这一点查看 Nginx 反向代理之后的 URL

{% asset_img Snipaste_2019-07-10_20-39-33.png %}

# 情景 1：proxy_pass 不带斜线

Nginx 的配置

```nginx
server {
    listen 80;
    server_name www.ikutarian.com;

    location /api {
        proxy_pass http://127.0.0.1:8080;  # 末尾没有斜线
    }
}
```

那么，当用户请求 `http://www.ikutarian.com/api/other` 时，匹配到该区块。Nginx 反向代理到后端时会保留虚拟路径，Nginx 实际向后端发起的 URL 为 `http://127.0.0.1/api/other`

## 情景 2：proxy_pass 有一个斜线

Nginx 的配置

```nginx
server {
    listen 80;
    server_name www.ikutarian.com;

    location /api {
        proxy_pass http://127.0.0.1:8080/;  # 末尾有一个斜线
    }
}
```

那么，当用户请求 `http://www.ikutarian.com/api/other` 时，匹配到该区块。由于 `proxy_pass` 中有指定后端 URI 的路径。Nginx 反向代理到后端时不会保留虚拟路径，Nginx 实际向后端发起的 URL 为 `http://127.0.0.1//other` **（注意：有两条斜线）**

# 场景 3：proxy_pass 有一个斜线，同时有一个 context-path

Nginx 的配置

```nginx
server {
    listen 80;
    server_name www.ikutarian.com;

    location /api {
        proxy_pass http://127.0.0.1:8080/ok;  # 末尾有一个斜线和一个 context-path
    }
}
```

那么，当用户请求 `http://www.ikutarian.com/api/other` 时，匹配到该区块。由于 `proxy_pass` 中有指定后端 URI 的路径。Nginx 反向代理到后端时不会保留虚拟路径，Nginx 实际向后端发起的 URL 为 `http://127.0.0.1/ok/other`

# 场景 4：proxy_pass 有一个斜线，同时有一个 context-path，末尾还有一个斜线

Nginx 的配置

```nginx
server {
    listen 80;
    server_name www.ikutarian.com;

    location /api {
        proxy_pass http://127.0.0.1:8080/ok/;  # 末尾有一个斜线和一个 context-path，末尾还有一个斜线
    }
}
```

那么，当用户请求 `http://www.ikutarian.com/api/other` 时，匹配到该区块。由于 `proxy_pass` 中有指定后端 URI 的路径。Nginx 反向代理到后端时不会保留虚拟路径，Nginx 实际向后端发起的 URL 为 `http://127.0.0.1/ok//other` **（注意：有两条斜线）**

# 总结

从上面三个场景可以发现，问题就在于 `proxy_pass` 指令中是否有指定后端服务器的 URI。这决定了 Nginx 反向代理到后端时是否会保留 `location` 中的虚拟路径

> 如果 proxy_pass **有指定 URI**，那么 Nginx 反向代理之后**不保留虚拟路径**

当后端服务器写成 `upstream` 方式时，效果也是一样的。比如

```nginx
upstream backend {
    server 127.0.0.1:8080;    
}


server {
    listen 80;
    server_name www.ikutarian.com;

    # 末尾没有斜线
    location /api {
        proxy_pass http://backend/ok;  # 末尾有一个斜线和一个 context-path
    }
}
```

当用户请求 `http://www.ikutarian.com/api/other` 时，Nginx 实际向后端发起请求的 URL 依然是 `http://127.0.0.1/ok/other`