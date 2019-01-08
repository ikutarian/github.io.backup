---
title: 基于Spring及Redis的Token鉴权
date: 2018-09-17 10:50:01
tags:
  - Token
  - Redis
  - SpringMVC
  - 鉴权
categories:
  - JavaWeb
---

## 为什么用 Token
一般来说都是用 session 来存储登录信息的，但是移动端使用 session 不太方便，所以一般都用 token 。另外现在前后端分离，一般都用 token 来鉴权。用 token 也更加符合 RESTful 中无状态的定义。

## 交互流程

1. 客户端通过登录请求提交用户名和密码，服务端验证通过后生成一个 Token 与该用户进行关联，并将 Token 返回给客户端。
2. 客户端在接下来的请求中都会携带 Token，服务端通过解析 Token 检查登录状态。
3. 当用户退出登录、其他终端登录同一账号（被顶号）、长时间未进行操作时 Token 会失效，这时用户需要重新登录。

<!-- more -->

## 程序示例

### Token的生成算法

服务端生成的 Token 一般为随机的非重复字符串，根据应用对安全性的不同要求，会将其添加时间戳（通过时间判断 Token 是否被盗用）或 url 签名（通过请求地址判断 Token 是否被盗用）后加密进行传输。因为只是个 demo，所以这里简单写了

```Java
public class TokenUtil {

    private static final String SEPARATOR = "-";

    /**
     * Token格式：时间戳-userId-随机字符串
     */
    public static String createToken(long userId) {
        return new Date().getTime() + SEPARATOR + userId + SEPARATOR + RandomStringUtils.random(10, true, true);
    }

    /**
     * 解析Token，从中取得userId
     */
    public static Long getUserIdFromToken(String token) {
        if (StringUtils.isEmpty(token)) {
            return null;
        }
        String[] param = token.split(SEPARATOR);
        if (param.length != 3) {
            return null;
        }
        try {
            return NumberUtils.createLong(param[1]);
        } catch (NumberFormatException e) {
            return null;
        }
    }
}
```

### Token的CRUD操作

Redis 是一个 Key-Value 结构的内存数据库，用它维护 userId 和 Token 的映射表会比传统数据库速度更快，这里使用 Spring-Data-Redis 封装的 RedisTokenManager 对 Token 进行基础操作：

首先定义一个 DAO 接口

```Java
package com.owen.favorite.repository;

import com.owen.favorite.domain.Token;

public interface TokenRepository {

    /**
     * 创建一个 token 并关联上指定用户
     */
    Token createToken(long userId);

    /**
     *  检查 token 是否有效
     */
    boolean checkToken(Token token);

    /**
     * 清除 token
     */
    void deleteToken (long userId);
}

```

然后是实现类

```Java
package com.owen.favorite.repository.impl;

import com.owen.favorite.constant.ApiConstant;
import com.owen.favorite.domain.Token;
import com.owen.favorite.repository.TokenDao;
import com.owen.favorite.util.TokenUtil;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Repository;

import java.util.concurrent.TimeUnit;

/**
 * 通过 Redis 管理 token 的实现类
 */
@Repository
public class RedisTokenDaoImpl implements TokenDao {

    private RedisTemplate<Long, String> redisTemplate;

    @Override
    public Token createToken(long userId) {
        String token = TokenUtil.createToken(userId);
        // 存储到 redis 并设置过期时间
        redisTemplate.boundValueOps(userId).set(token, ApiConstant.Token.EXPIRE_DAYS, TimeUnit.DAYS);
        return new Token(userId, token);
    }

    @Override
    public boolean checkToken(String tokenFromClient) {
        if (StringUtils.isEmpty(tokenFromClient)) {
            return false;
        }
        Long userId = TokenUtil.getUserIdFromToken(tokenFromClient);
        if (userId == null) {
            return false;
        }
        String tokenInRedis = redisTemplate.boundValueOps(userId).get();
        if (tokenFromClient.equals(tokenInRedis)) {
            // 如果验证成功，说明此用户进行了一次有效操作，延长 token 的过期时间
            redisTemplate.boundValueOps(userId).expire(ApiConstant.Token.EXPIRE_DAYS, TimeUnit.DAYS);
            return true;
        } else {
            return false;
        }
    }

    @Override
    public void deleteToken(long userId) {
        redisTemplate.delete(userId);
    }

    @Autowired
    public void setRedisTemplate(RedisTemplate<Long, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }
}
```

## 登录与注册

登录与注册的 Controller

```Java
package com.owen.favorite.controller;

import com.owen.favorite.domain.APIResult;
import com.owen.favorite.domain.Token;
import com.owen.favorite.service.TokenService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Autowired
    private TokenService tokenService;

	@PostMapping("/login")
	public APIResult login(@RequestParam String username, @RequestParam String password) {
        User user = userService.findByUsername(username);
        if (user == null /* 未注册 */ || !user.getPassword().equals(password) /* 密码错误 */) {
            return APIResult.createNg("用户名或密码错误");
        }
        //生成一个token，保存用户登录状态
        Token token = tokenService.createToken(user.getId());
        return APIResult.createOk(token);
    }

    @PostMapping("/logout")
    public APIResult logout(@RequestParam String token) {
        Long userId = TokenUtil.getUserIdFromToken(token);
        if (userId == null) {
            return APIResult.createNg("退出失败");
        }
        tokenService.deleteToken(userId);
        return APIResult.createOKMessage("退出成功");
    }
}
```

### token验证

客户端访问一些需要用户登录之后才能调用的接口，比如在数据库中插入一条记录，那么就需要判断 token 的合法性。而这样的接口又有很多，那么岂不是每一次都需要及你想那个判断，代码要重复写很多遍。这时候可以使用自定义注解和拦截器来实现。

首先定义一个注解

```Java
package com.owen.favorite.anno;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 在 Controller 的方法上使用此注解，该方法在映射时会检查用户是否登录，未登录返回 401 错误
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Authorization {
}
```

拦截器的实现

```Java
package com.owen.favorite.interceptor;

import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.owen.favorite.anno.Authorization;
import com.owen.favorite.constant.ApiConstant;
import com.owen.favorite.domain.APIResult;
import com.owen.favorite.service.TokenService;
import com.owen.favorite.util.TokenUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.PrintWriter;

/**
 * 自定义拦截器，判断此次请求的用户是否已登录
 */
@Component
public class AuthorizationInterceptor extends HandlerInterceptorAdapter {

    @Autowired
    private TokenService tokenService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (!(handler instanceof HandlerMethod)) {
            // 如果不是映射到方法直接通过
            return true;
        }
        //从header中得到token
        String token = request.getHeader(ApiConstant.RequestParam.TOKEN);
        // 验证 token
        if (tokenService.checkToken(token)) {
            //如果token验证成功，将token对应的用户id存在request中，便于之后注入
            request.setAttribute(ApiConstant.RequestParam.USER_ID, TokenUtil.getUserIdFromToken(token));
            return true;
        } else {
            // 如果验证token失败，并且方法注明了Authorization，就告诉客户端token不对
            HandlerMethod handlerMethod = (HandlerMethod) handler;
            if (handlerMethod.getMethodAnnotation(Authorization.class) != null) {
                response.setCharacterEncoding("UTF-8");
                response.setContentType("application/json;charset=utf-8");

                ObjectMapper objectMapper = new ObjectMapper();
                objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

                PrintWriter writer = response.getWriter();
                writer.write(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(APIResult.createNg("请登录")));
                writer.close();
                return false;
            } else {
                return true;
            }
        }
    }
}
```

## 一些细节

* 登录请求一定要使用 HTTPS，否则无论 Token 做的安全性多好密码泄露了也是白搭
* Token 的生成方式有很多种，例如比较热门的有 JWT（JSON Web Tokens）、OAuth 等。

