---
title: 利用SpringSession实现Tomcat集群的会话管理
date: 2019-04-08 23:04:56
tags:
  - SpringBoot
  - SpringSession
  - Session
  - Tomcat
  - 集群
categories:
  - SpringBoot
---

后端使用 Tomcat 集群的话，Session 的管理是一个问题，不过利用 Spring Session 来管理的话就很简单

<!-- more -->

# 架构

利用 Nginx 的反向代理，把请求转发到 Tomcat 集群，Tomcat 集群通过 Spring Session 访问 Redis 读写会话信息

# Session 的默认实现

默认情况下，SpringBoot 使用 Tomcat 的 Session 实现。用一个例子来验证一下

```java
@Controller
public class SpringSessionController {

    private static final Logger log = LoggerFactory.getLogger(SpringSessionController.class);

    @RequestMapping("session")
    @ResponseBody
    public String session(HttpServletRequest request) {
        HttpSession session = request.getSession();

        log.info("class = {}", session.getClass());
        log.info("id = {}", session.getId());

        String name = "ikutarian";
        session.setAttribute("user", name);
        return "hey, " + name;
    }
}
```

访问 `http://127.0.0.1:8080/session`，控制台输出

```
class = class org.apache.catalina.session.StandardSessionFacade
id = E21BC48F9C84B4A8EA0C9303767682A8
```

可以看到 `HttpSession` 的实现类是 `org.apache.catalina.session.StandardSessionFacade`

# Spring Session 介绍

https://docs.spring.io/spring-session/docs/current/reference/html5/

# Spring Session 的集成

以下过程参考了官网的[HttpSession with Redis Guide](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/boot-redis.html)

加入两个依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
```

或者一步到位，只加入一个依赖即可（因为 `spring-session-data-redis` 关联了 `spring-boot-starter-data-redis` 和 `spring-session`）

```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

在 `application.yml` 加入以下配置

```yml
spring:
  session:
    # Session 的存储方式
    store-type: redis 
  redis:
    # redis 连接信息
    host: localhost
    port: 6479
    password: 123@456
```

再次运行上面的例子，控制台输出

```
class = class org.springframework.session.web.http.SessionRepositoryFilter$SessionRepositoryRequestWrapper$HttpSessionWrapper
id = 8565d8f2-f3af-4772-b73a-efa2d1a62388
```

可以看到现在 `HttpSession` 的实现类变了，并且 Redis 里多了一些以 `spring:session` 开头的 key

```
127.0.0.1:6379> keys *
1) "spring:session:expirations:1554733860000"
2) "spring:session:sessions:8565d8f2-f3af-4772-b73a-efa2d1a62388"
3) "spring:session:sessions:expires:8565d8f2-f3af-4772-b73a-efa2d1a62388"
```

# 反向代理的配置

现在准备两个配置好 Spring Session 的 SpringBoot 应用，分别运行在 8080 和 8081 端口上。接着打开 Nginx 的配置文件，在 `http` 指令内添加如下配置信息

```nginx
# Tomcat 集群
upstream ikutarian {
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}

server {
    # 监听的端口
    listen 80;
    # 域名
    server_name localhost;

    location / {
        # 请求转发到 Tomcat 集群
        proxy_pass  http://ikutarian/;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

现在访问 `http://localhost/session`，观察 8080 和 8081 的输出，可以看到 Session 的 ID 都是一样的，这说明成功实现了 Session 的分布式管理