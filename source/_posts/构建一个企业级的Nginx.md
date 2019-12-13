---
title: 构建一个企业级的nginx
date: 2019-07-17 09:25:07
tags:  
  - nginx
  - 安全
  - 企业级
categories:
  - nginx
---

自己使用 nginx 的时候，都是用最简配置。如果是企业应用场景，应该怎么做？

<!-- more -->

# 前言

可以从一下几个方面入手

## 一、安全方面

1. 隐藏 nginx header 版本号
2. 更改源码，隐藏 nginx 软件名称
3. 更改 nginx 默认用户及用户组
4. nginx 站点目录及文件 URL 访问控制（防止恶意解析）
5. 防止恶意解析访问企业网站
6. 配置 nginx 图片及目录防盗链
7. 配置 nginx 防爬虫
8. 限制 HTTP 请求方法
9. 防 DDOS 攻击

## 二、性能方面

1. 根据 CPU 逻辑核心数，配置 nginx worker 进程个数
2. 配置 nginx worker 进程的 CPU 亲和力参数
3. 配置 ngixn worker 单个进程允许的客户端最大连接数
4. 配置 nginx worker 进程的最大打开文件数
5. 配置 nginx 事件处理模型为 epoll

## 三、HTTP 协议方面

1. 开启高效的文件传输模式（sendfile/tcp_nopush/tcp_nodelay）
2. 设置连接超时时间
3. 设置客户端上传文件大小
4. fastcgi 调优

## 四、功能方面

1. 配置 nginx gzip 压缩功能
2. 配置 nginx expires 缓存功能
3. 配置 nginx 错误页面的优雅显示

## 五、日志方面

1. 每天进行日志切割、备份/不记录不需要的访问日志/访问日志的权限设置

## 六、架构方面

1. nginx 程序架构优化(服务解耦)
2. 使用 CDN 为网站内容加速

接下来进行具体说明

# 一、安全方面

## 1. 隐藏 nginx header 版本号

没有隐藏版本号时，使用 Chrome 浏览器访问 nginx，可以在开发者工具中看到 `Header` 的 `Server` 的内容为

{% asset_img Snipaste_2019-07-17_10-38-50.png %}

不同版本的 nginx 有不同的漏洞，如果知道了版本号，容易让别人根据版本对 nginx 进行攻击，因此应该隐藏 nginx 的版本号

打开 `nginx.conf`，在 `http` 部分添加以下内容

```nginx
# 省略...

http {
  # 省略...

  server_tokens off;

  # 省略...
}

# 省略...
```

然后重启 nginx，再次使用 Chrome 浏览器访问 nginx。可以看到 nginx 的版本号已经没有了

{% asset_img Snipaste_2019-07-17_10-44-05.png %}

## 2. 更改源码，隐藏 nginx 软件名称

隐藏 nginx 的版本号还是不够彻底，如果把 nginx 的名称改成其他的，可以更加迷惑黑客。把软件名改成 Apache 或者 IIS 都行，不过 IIS 更好。因为 IIS 只能安装在 Windows 平台。那么入侵者看到了 IIS 就以为服务器是 Windows 系统，那么它使用的漏洞工具就只针对 Windows，但实际上服务器是 Linux，这样黑客就更迷惑了

打开 `src/core/nginx.h`，修改 nginx 的版本号和版本名称，假装是 `Microsoft-IIS/8.5`

```c
#define nginx_VERSION      "8.5"
#define nginx_VER          "Microsoft-IIS/" nginx_VERSION
```

然后再重新编译安装 nginx。现在访问 nginx 之后，`Header` 中的 `Server` 内容变了

{% asset_img Snipaste_2019-07-17_11-20-55.png %}

## 3. 更改 nginx 默认用户及用户组

不要用 root 权限来运行 nginx，否则黑客拿到的 WebShell 权限很可能是 root 权限

创建用户组

```
$ groupadd nginx
```

创建一个用户，禁止ssh登录，且不创建 home 目录

```
$ useradd -s /sbin/nologin -M nginx
```

然后修改 `nginx.conf`

```nginx
user nginx nginx; # 前面是用户，后面是用户组
```

在编译安装的时候也可以指定用户与用户组

```
$ ./configure --prefix=/usr/local/nginx-1.12.2 --user=nginx --group=nginx
```

现在运行 `ps aux | grep nginx` 查看 nginx 的进程信息

```
root@ikutarian:~# ps aux | grep nginx
root       8873  0.0  0.0  24928   412 ?        Ss   13:38   0:00 nginx: master process ./nginx
nginx      8874  0.0  0.1  25384  2520 ?        S    13:38   0:00 nginx: worker process
```

可以看到 Nnginx 的 `worker` 进程的用户是 `nginx`

4. nginx 站点目录及文件 URL 访问控制（防止恶意解析）
5. 防止恶意解析访问企业网站
6. 配置 nginx 图片及目录防盗链
7. 配置 nginx 防爬虫
## 8. 限制 HTTP 请求方法

使用 [limit_except](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_except) 指令来实现

```nginx
server {
    // 省略
    
    location /api {
        # 只允许 GET 和 POST 请求，其他的请求直接返回 403 状态码
        limit_except GET POST {
            deny  all;
        }
    
        proxy_pass http://127.0.0.1:8090/api;
    }	
}
```

注意：允许了 `GET` 请求，意味着也允许了 `HEAD` 请求

9. 防 DDOS 攻击

nginx