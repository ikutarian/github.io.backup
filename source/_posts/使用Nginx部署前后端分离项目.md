---
title: 使用Nginx部署前后端分离项目
date: 2019-07-10 19:46:25
tags:  
  - Nginx
  - 前后端分离
  - SpringBoot
  - location
  - proxy_pass
categories:
  - Nginx
---

现在的趋势是前后端分离，正好工作中有用到，因此总结一下

<!-- more -->

# 环境

- nginx-1.14.0
- Windows 10

# Nginx的配置文件

因为是 Windows 版本的 Nginx，所以配置文件位置是 `conf/` 下的 `nginx.conf`

在 `http` 部分的最底部加入一条配置

```nginx
# 省略...

http {

    # 省略...
    
    include vhost/*.conf;
}
```

表示在 Ngninx 的 `conf/` 目录下的 `vhost/` 文件夹内，所有以 `conf` 为后缀的文件都是 Nginx 配置文件。一个域名一个 `conf` 配置文件，这样做的目的是方便维护

比如，资源文件的域名是 `file.ikutarian.com`，后端的域名是 `api.ikutarian.com`，那么 `vhost/` 文件夹下就有了这两个 Nginx 配置文件，它们分别配置这两个域名的信息，方便维护

- file.ikutarian.com.conf
- api.ikutarian.com.conf

# 约定

- Nginx 所在服务器的 IP 是 `192.168.168.100`，开放 `80` 端口
- 前端静态页面的存放位置是 `F:\\front`，首页是 `login.html`
- 后端的 BaseURL 是 `http://192.168.168.100:2077/api/`

# 配置

在 `conf/vhost/` 文件夹下新建一个名为 `192.168.168.100.conf` 的文件

```nginx      
server {
    # 监听的端口
    listen       80;
    # 服务器的IP地址
    server_name  192.168.168.100;
    
    # 静态页面
    location / {
        # 存放的位置
        root   F:\\front;
        # 首页
        index  login.html;
    }
    
    # 以api前缀开头的请求都转发到后端
    location /api {
        proxy_pass http://192.168.168.100:2077/api;
    }
}
```

# 启动Nginx

首先检测配置文件是否正确

```
$ nginx -t
```

然后启动 Nginx 

```
$ start nginx
```

# location与proxy_pass的匹配规则

具体看这篇文章 {% post_link Nginx中location与proxy-pass的匹配规则 %}