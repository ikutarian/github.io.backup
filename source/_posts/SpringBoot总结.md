---
title: SpringBoot总结
date: 2018-10-31 23:15:57
tags:
  - SpringBoot
  - 总结
categories:
  - SpringBoot
---

本文是我学习与使用 SpringBoot 的总结，记录下来，以免忘记。

<!-- more -->

## 项目的初始化

可以在[Spring Initializr](https://start.spring.io/)上进行配置，然后初始化一个 SpringBoot 项目，下载加载导入 IDEA 即可

只需要填写以下信息：

- 构建方式，比如 Maven
- 使用的语言，比如 Java
- SpringBoot 的版本
- Group 与 Artifact
- 依赖，比如 web

{% asset_img QQ截图20181101102056.png %}

下载下来的是一个 zip 压缩包，解压放到硬盘的某一个位置。有几个文件没有用，可以删掉：

- .mvn/
- mvnw
- mvnw.cmd

项目的文件结构和 Maven 规定的文件结构是一样的，只是 `resources` 文件夹有额外的内容，打开就可以看到：

- static/
- templates/
- application.properties

`static/` 用来放静态资源文件，`templates/` 用来放模板文件，`application.properties` 是项目的属性配置文件

## 配置文件

SpringBoot 支持两种配置文件类型：

- application.properties
- application.yml

`application.properties` 是默认的的属性配置文件。推荐使用 `application.yml`，因为更容易阅读。配置要怎么写，可以看 [common-application-properties](https://docs.spring.io/spring-boot/docs/1.5.17.RELEASE/reference/htmlsingle/#common-application-properties)

举个例子，我要配置 ContextPath 和内置 Tomcat 的端口

```yml
server:
  port: 1234
  context-path: /demo
```

访问 http://localhost:1234/demo/ 就可以看到返回的信息了。因为现在没有写代码，所以返回的是一个错误页面。不过这也说明了这个 URL 是可以访问到项目的

{% asset_img QQ截图20181101105921.png %}

也可以在配置文件中写自定义的配置信息，然后在代码中获取到配置信息

```yml
projectName: demo

developer:
  name: ikutarian
  age: 18
```

对于 `project_name`，在 Java 代码中可以利用 `@Value` 注解获取到

```java
@Value("${projectName}")
private String projectName;
```

对于 `developer` 需要创建一个类

```java
@Component
@ConfigurationProperties(prefix = "developer")
public class Developer {

    private String name;
    private Integer age;

    // getter
    // setter
}
```

还要加入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

然后要获取 Developer 的信息是，只需要自动装配即可

```java
@Autowired
private Developer developer;
```

一个例子如下：

```java
@RestController
public class HelloController {

    @Value("projectName")
    private String projectName;

    @Autowired
    private Developer developer;

    @RequestMapping("hello")
    public String hello() {
        return "projectName=" + projectName + "\n" +
                developer.toString();
    }
}
```

## 配置文件管理

可以写多个类似 `application-{profile}.yml` 格式的配置文件，然后指定使用哪一个

首先项目中还是必须得有 `application.yml`，以它为主。然后可以按照 `application-{profile}.yml` 格式写其他的配置文件

假设，现在加上两个配置文件：

- application-dev.yml 开发环境的配置
- application-prod.yml 生产环境用的配置

application-dev.yml

```yml
projectName: dev

developer:
  name: ikutarian-dev
  age: 18
```

application-prod.yml

```yml
projectName: prod

developer:
  name: ikutarian-prod
  age: 19
```

然后在 `application.yml` 中指定使用哪一个配置，现在指定使用 `application-dev.yml`

```yml
spring:
  profiles:
    active: dev
```

如果要指定 `application-prod.yml`，只需要改成这样即可

```yml
spring:
  profiles:
    active: prod
```

也可以在部署 jar 包时指定使用哪一个配置

```
java -jar xxx.jar --spring.profiles.active=prod
```