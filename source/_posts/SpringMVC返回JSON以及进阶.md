---
title: SpringMVC返回JSON以及进阶
date: 2018-09-19 09:12:17
tags:
  - JSON
  - Jackson
  - SpringMVC
  - SpringBoot
categories:
  - JavaWeb
---

## 运行效果

{% asset_img 3617116-648b6a99eb73396e.png %}

<!-- more -->

## 开发环境

1. IDEA 2017
2. JDK 1.8
3. Spring 4.2.2.RELEASE
4. Jackson 2.8.5

## 项目结构

```
└─main
    ├─java
    │  └─com
    │      └─smart
    │          ├─controller
    │          │      JSONController.java
    │          │
    │          └─domain
    │                  User.java
    │
    ├─resources
    │      smart-context.xml
    │
    └─webapp
        └─WEB-INF
                web.xml

```

因为是一个小 demo，所以代码尽量简单，只有一个 Controller 和 Domain 对象

## 依赖

`pom.xml`

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.smart</groupId>
    <artifactId>chapter2</artifactId>
    <packaging>war</packaging>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.encoding>UTF-8</maven.compiler.encoding>

        <spring.version>4.2.2.RELEASE</spring.version>
        <jackson.version>2.8.5</jackson.version>
        <tomcat7-maven-plugin>2.2</tomcat7-maven-plugin>
    </properties>

    <dependencies>
        <!-- Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>${jackson.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>${tomcat7-maven-plugin}</version>
                <configuration>
                    <path>/${project.artifactId}</path>
                    <uriEncoding>UTF-8</uriEncoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

只需要 SpringMVC 和 Jackson 依赖即可

## Domain

```Java
package com.smart.domain;

public class User {

    private String userName;
    private String password;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## Controller

```Java
package com.smart.controller;

import com.smart.domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

@Controller
public class JSONController {

    @RequestMapping("/testJavaBean")
    @ResponseBody
    public User test(@RequestParam("name") String name) {
        User user = new User();
        user.setUserName(name);
        user.setPassword("123456");
        return user;
    }


    @RequestMapping("/testMap")
    @ResponseBody
    public Map test2(@RequestParam("name") String name) {
        Map<String, Object> map = new HashMap<>();
        map.put("name", name);
        map.put("test", 123);
        map.put("array", new String[]{"a", "b", "c"});
        return map;
    }
}
```

别忘了给 Controller 的方法添加上 `@ResponseBody` 注解，这样才能返回 JSON

## ApplicationContext.xml

`smart-context.xml`

```XML
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.smart.controller"/>

    <mvc:annotation-driven/>

</beans>
```

Spring 的配置文件也是很简单，关键在于要加上 `<mvc:annotation-driven/>`

## web.xml

```XML
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:smart-context.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>smart</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:smart-context.xml</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>smart</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```

在 `web.xml` 指定 ApplicationContext 的文件位置即可

## 运行结果

URL `http://localhost:8080/chapter2/testJavaBean?name=我是中文`

{% asset_img 3617116-648b6a99eb73396e.png %}

## 进阶

一般在真实开发的时候，JSON 数据一般是这样的

* 请求成功的时

```
{
    "code": 200,
    "data": {
        "userId": 1,
        "userName": "Admin",
        "credits": 160,
        "lastIp": "0:0:0:0:0:0:0:1",
        "lastVisit": 1514968628552
    }
}
```

* 请求失败时

```
{
    "code": 404,
    "message": "账号或密码错误"
}
```

从上面可以看到，服务器传回来的 JSON 格式是这样

```
{
	"code": xx,
	"message": xx,
    "data": xxx
}
```

而且，如果 JSON 数据中的值为 `null` 的话，就不传回来。对于这样的需求要怎么实现呢？

### APIResult

首先按照服务器的 JSON 格式，定义一个 JavaBean，至于 `data` 部分用 `Object` 表示

```Java
package com.smart.domain;

import com.smart.constant.ApiConstant;

public class APIResult {

    private int code;
    private String message;
    private Object data;

    public static APIResult createOk(Object data) {
        return createWithCodeAndData(ApiConstant.Code.OK, null, data);
    }

    public static APIResult createOKMessage(String message) {
        APIResult result = new APIResult();
        result.setCode(ApiConstant.Code.OK);
        result.setMessage(message);
        return result;
    }

    public static APIResult createNg(String message) {
        return createWithCodeAndData(ApiConstant.Code.NG, message, null);
    }

    private static APIResult createWithCodeAndData(int code, String message, Object data) {
        APIResult result = new APIResult();
        result.setCode(code);
        result.setMessage(message);
        result.setData(data);
        return result;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }
}
```

对于 `null` 字段不显示的需求，只需要在 ApplicationContext 配置文件定义即可。注意了，这个定义是全局生效的

```XML
<bean id="objectMapper" class="com.fasterxml.jackson.databind.ObjectMapper">
	<property name="serializationInclusion" value="NON_NULL"/>
</bean>

<mvc:annotation-driven>
	<mvc:message-converters>
		<bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
			<property name="objectMapper" ref="objectMapper" />
			<property name="prettyPrint" value="true"/>
		</bean>
	</mvc:message-converters>
</mvc:annotation-driven>
```

最后就是修改一下 Controller 的代码，使用 `APIResult` 将数据包装起来

```Java
package com.smart.controller;

import com.smart.domain.APIResult;
import com.smart.domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

@Controller
public class JSONController {

    @RequestMapping("/testJavaBean")
    @ResponseBody
    public APIResult test(@RequestParam("name") String name) {
        User user = new User();
        user.setUserName(name);
        user.setPassword("123456");
        return APIResult.createOk(user);
    }


    @RequestMapping("/testMap")
    @ResponseBody
    public APIResult test2(@RequestParam("name") String name) {
        Map<String, Object> map = new HashMap<>();
        map.put("name", name);
        map.put("test", 123);
        map.put("array", new String[]{"a", "b", "c"});
        return APIResult.createOk(map);
    }

    @RequestMapping("/testNg")
    @ResponseBody
    public APIResult testNg() {
        return APIResult.createNg("用户名或密码错误");
    }
}
```

URL: http://localhost:8080/chapter2/testNg

返回

```
{
    "code": 404,
    "message": "用户名或密码错误"
}
```

URL http://localhost:8080/chapter2/testMap?name=Admin

返回

```
{
    "code": 200,
    "data": {
        "test": 123,
        "array": [
            "a",
            "b",
            "c"
        ],
        "name": "Admin"
    }
}
```

URL http://localhost:8080/chapter2/testJavaBean?name=Admin

返回

```
{
    "code": 200,
    "data": {
        "userId": 0,
        "userName": "Admin",
        "password": "123456",
        "credits": 0
    }
}
```

## 对应SpringBoot框架的做法
如果是 SpringBoot 框架来开发的话，更简单了，不需要写一堆的配置文件。

### Application

```Java
package com.smart;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.web.SpringBootServletInitializer;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@SpringBootApplication
@EnableTransactionManagement
public class Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder applicationBuilder) {
        return applicationBuilder.sources(Application.class);
    }
}
```

只需要一个使用 `@SpringBootApplication` 讲某一个类注解为启动类即可，不需要写 ApplicationContext.xml 了

## Controller

相反，Controller 只需要使用 `@RestController` 注解即可，可以将方法上的 `@ResponseBody` 注解去掉

```Java
package com.smart.controller;

import com.smart.domain.APIResult;
import com.smart.domain.User;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class JSONController {

    @RequestMapping("/testJavaBean")
    public APIResult test(@RequestParam("name") String name) {
        User user = new User();
        user.setUserName(name);
        user.setPassword("123456");
        return APIResult.createOk(user);
    }

    @RequestMapping("/testNg")
    public APIResult testNg() {
        return APIResult.createNg("用户名或密码错误");
    }
}
```

对于JSON的格式要求：
1. 去除 Null 字段
2. 格式化打印

只需要在 `src/main/resources/` 文件夹中创建一个 `application.properties` 文件，添加如下内容

```
spring.jackson.serialization.indent-output=true
spring.jackson.serialization-inclusion=non_null
```
