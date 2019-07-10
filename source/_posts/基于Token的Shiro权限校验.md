---
title: 基于Token的Shiro权限校验
date: 2019-07-08 09:49:52
tags:  
  - Token
  - Shiro
  - 权限校验
  - 前后端分离
  - SpringBoot
categories:
  - Shiro
---

如何用 Shiro 搭建一个基于 Token 的权限校验系统？

<!-- more -->

# Shiro的架构

{% asset_img 1.png %}

- `Authentication`：身份验证，检测用户的账号密码是否正确
- `Authorization`：权限验证，验证用户是否有某一个角色/权限

# Shiro的组件

{% asset_img 5bf540925c29b.png %}

- `SecurityManager`：安全管理器，即所有跟安全有关的操作都会与 `SecurityManager` 交互。且它管理者所有 `Subject`。它是 Shiro 的核心
- `Subject`：主体，代表了当前的“用户”，所有 `Subject` 都绑定到 `SecurityManager`
- `Realm`：Shiro 从 `Realm` 中获取安全数据（如用户、角色、权限），就是说 `SecurityManager` 如果要验证用户的身份或者角色/权限，那么它需要从 `Realm` 中获取相应用户进行校验。可以把 `Realm` 看成是 DataSource。`Reaml` 需要由用户自己实现

# 用 Shiro 搭建一个基于 Token 的权限校验系统

## 身份令牌

身份验证，即在应用中谁能证明他就是他的本人。在我们搭建的基于 Token 的认证系统当中，身份验证标识当然是根据一定规则生成的 Token 了。我们定义 `OAuth2Token`，让它实现 `org.apache.shiro.authc.AuthenticationToken` 接口

接口提供了两个方法

```java
public interface AuthenticationToken extends Serializable {

    /**
     * 用于获得主体的标识属性，可以是用户名
     */
    Object getPrincipal();

    /**
     * 用于获得证明/凭证，如密码、证书等
     */
    Object getCredentials();
}
```

Shiro 有一个默认的接口实现 `UsernamePasswordToken`，它将用户名当做是标识属性，密码当做是凭证

```java
public class UsernamePasswordToken implements HostAuthenticationToken, RememberMeAuthenticationToken {

    // 省略...

    public Object getPrincipal() {
        return getUsername();
    }

    public Object getCredentials() {
        return getPassword();
    }

    // 省略...
}
```

现在我们要实现的是基于 Token 的校验，自定义的 `AuthenticationToken` 实现如下

```java
package io.renren.modules.sys.oauth2;

import org.apache.shiro.authc.AuthenticationToken;

public class OAuth2Token implements AuthenticationToken {

    private String token;

    public OAuth2Token(String token){
        this.token = token;
    }

    @Override
    public String getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}
```

## 自定义 Reaml

`Realm` 用来提供安全数据源，即校验用户身份验证是否通过，以及提供该用户相应的角色和权限的数据。注意，校验用户是否有正确的权限不是有 `Realm` 来处理，`Realm` 是负责提供数据，而交给 `SecurityManager` 来处理

自定义一个继承 `org.apache.shiro.realm.AuthorizingRealm` 的类，并重写 `doGetAuthorizationInfo` 和 `doGetAuthenticationInfo` 方法。还要重写 `supports` 方法，该方法的返回值（True、False）表明是否支持处理传入的 `AuthenticationToken` 类型的令牌

```java
package io.renren.modules.sys.oauth2;

import io.renren.modules.sys.entity.SysUserEntity;
import io.renren.modules.sys.entity.SysUserTokenEntity;
import io.renren.modules.sys.service.ShiroService;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.util.Set;

@Component
public class OAuth2Realm extends AuthorizingRealm {

    @Autowired
    private ShiroService shiroService;

    @Override
    public boolean supports(AuthenticationToken token) {
        // 只支持OAuth2Token令牌类型
        return token instanceof OAuth2Token;
    }

    /**
     * 授权(验证权限时调用)
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        SysUserEntity user = (SysUserEntity)principals.getPrimaryPrincipal();
        Long userId = user.getUserId();

        //用户权限列表
        Set<String> permsSet = shiroService.getUserPermissions(userId);

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        info.setStringPermissions(permsSet);

        return info;
    }

    /**
     * 认证(登录时调用)
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String accessToken = (String) token.getPrincipal();

        // 根据accessToken，从数据库查询用户信息
        SysUserTokenEntity tokenEntity = shiroService.queryByToken(accessToken);
        // token失效
        if(tokenEntity == null || tokenEntity.getExpireTime().getTime() < System.currentTimeMillis()){
            throw new IncorrectCredentialsException("token失效，请重新登录");
        }

        // 查询用户信息
        SysUserEntity user = shiroService.queryUser(tokenEntity.getUserId());
        // 账号锁定
        if(user.getStatus() == 0){
            throw new LockedAccountException("账号已被锁定,请联系管理员");
        }

        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, accessToken, getName());
        return info;
    }
}
```

## 自定义 Filter

```java
package io.renren.modules.sys.oauth2;

import com.google.gson.Gson;
import io.renren.common.utils.HttpContextUtils;
import io.renren.common.utils.R;
import org.apache.commons.lang.StringUtils;
import org.apache.http.HttpStatus;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.web.filter.authc.AuthenticatingFilter;
import org.springframework.web.bind.annotation.RequestMethod;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * oauth2过滤器
 */
public class OAuth2Filter extends AuthenticatingFilter {

    @Override
    protected AuthenticationToken createToken(ServletRequest request, ServletResponse response) {
        // 获取请求token
        String token = getRequestToken(request);

        // 没有token就返回null
        if(StringUtils.isBlank(token)){
            return null;
        }

        // 否则创建一个OAuth2Token对象并返回
        return new OAuth2Token(token);
    }

    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        // 如果是跨域请求时用到的HTTP OPTIONS请求，那就就放过，否则需要进行权限校验
        // 如果返回false，就会调用onAccessDenied方法
        return httpRequest.getMethod().equals(RequestMethod.OPTIONS.name());

    }

    /**
     * isAccessAllowed返回false时，会调用这个方法
     */
    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
        // 获取请求token
        String token = getRequestToken(request);

        // 如果token不存在，那就告知前端发生了401错误
        if(StringUtils.isBlank(token)){
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setHeader("Access-Control-Allow-Credentials", "true");
            httpResponse.setHeader("Access-Control-Allow-Origin", HttpContextUtils.getOrigin());

            String json = new Gson().toJson(R.error(HttpStatus.SC_UNAUTHORIZED, "invalid token"));
            httpResponse.getWriter().print(json);
            return false;
        }

        // 去执行subject.login(token)的登陆逻辑，subject.login(token)会去调用自定义Ream的doGetAuthenticationInfo方法
        return executeLogin(request, response);
    }

    /**
     * subject.login(token)失败的时候，会调用这个方法
     */
    @Override
    protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request, ServletResponse response) {
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        httpResponse.setContentType("application/json;charset=utf-8");
        httpResponse.setHeader("Access-Control-Allow-Credentials", "true");
        httpResponse.setHeader("Access-Control-Allow-Origin", HttpContextUtils.getOrigin());
        try {
            // 给前端返回登陆失败的具体信息
            Throwable throwable = e.getCause() == null ? e : e.getCause();
            R r = R.error(HttpStatus.SC_UNAUTHORIZED, throwable.getMessage());
            String json = new Gson().toJson(r);
            httpResponse.getWriter().print(json);
        } catch (IOException e1) {
        }

        return false;
    }

    // token的参数名称
    private static final String TOKEN_NAME = "token";

    /**
     * 获取请求的token
     */
    private String getRequestToken(ServletRequest request){
        HttpServletRequest httpRequest = (HttpServletRequest) request;

        // 从header中获取token
        String token = httpRequest.getHeader(TOKEN_NAME);

        //如果header中不存在token，则从参数中获取token
        if(StringUtils.isBlank(token)){
            token = httpRequest.getParameter(TOKEN_NAME);
        }

        return token;
    }
}
```

## 与 SpringBoot 集成

```java
package io.renren.config;

import io.renren.modules.sys.oauth2.OAuth2Filter;
import io.renren.modules.sys.oauth2.OAuth2Realm;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import javax.servlet.Filter;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.Map;

/**
 * Shiro配置
 */
@Configuration
public class ShiroConfig {

    /**
     * 生成DefaultWebSecurityManager bean，并设置我们自定义的Realm
     * @param oAuth2Realm Spring将会自动注入MyRealm类，因为我们给它加了@Component注解
     */
    @Bean("securityManager")
    public SecurityManager securityManager(OAuth2Realm oAuth2Realm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(oAuth2Realm);
        securityManager.setRememberMeManager(null);
        return securityManager;
    }

    /**
     * 注入ShiroFilter，配置过滤的URL规则
     */
    @Bean("shiroFilter")
    public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager);

        // 添加OAuth2Filter过滤器且命名为oauth2
        Map<String, Filter> filters = new HashMap<>();
        filters.put("oauth2", new OAuth2Filter());
        shiroFilter.setFilters(filters);

        Map<String, String> filterMap = new LinkedHashMap<>();
        filterMap.put("/webjars/**", "anon");
        filterMap.put("/druid/**", "anon");
        filterMap.put("/app/**", "anon");
        filterMap.put("/sys/login", "anon");
        filterMap.put("/swagger/**", "anon");
        filterMap.put("/v2/api-docs", "anon");
        filterMap.put("/swagger-ui.html", "anon");
        filterMap.put("/swagger-resources/**", "anon");
        filterMap.put("/captcha.jpg", "anon");
        filterMap.put("/**", "oauth2");
        shiroFilter.setFilterChainDefinitionMap(filterMap);

        return shiroFilter;
    }

    // --------- 开启Shiro的注解功能  -------------
    @Bean("lifecycleBeanPostProcessor")
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
    // --------- 开启Shiro的注解功能  -------------
}
```

# 使用

## API接口权限控制

```java
package io.renren.modules.sys.controller;

import io.renren.common.utils.PageUtils;
import io.renren.common.utils.R;
import io.renren.modules.sys.service.SysLogService;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import java.util.Map;

/**
 * 系统日志
 */
@RestController
@RequestMapping("sys/log")
public class SysLogController {

	@Autowired
	private SysLogService sysLogService;
	
	/**
	 * 列表
	 */
	@GetMapping("list")
	@RequiresPermissions("sys:log:list")
	public R list(@RequestParam Map<String, Object> params){
		PageUtils page = sysLogService.queryPage(params);
		return R.ok().put("page", page);
	}
}
```

通过 `@RequiresPermissions("sys:log:list")` 来校验用户是否有权限调用这个接口

## 捕捉Shiro的异常

在全局异常处理器中进行处理

```java
package io.renren.common.exception;

import io.renren.common.utils.R;
import org.apache.shiro.authz.AuthorizationException;

/**
 * 异常处理器
 */
@RestControllerAdvice
public class RRExceptionHandler {
    
	@ExceptionHandler(AuthorizationException.class)
	public R handleAuthorizationException(AuthorizationException e){
		logger.error(e.getMessage(), e);
		return R.error("没有权限，请联系管理员授权");
	}
}
```