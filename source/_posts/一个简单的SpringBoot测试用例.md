---
title: 一个简单的SpringBoot测试用例
date: 2018-12-23 17:13:24
tags:
  - SpringBoot
  - 测试
  - MockMvc
categories:
  - SpringBoot
---

在 SpringBoot 中如何测试 Web Layer

<!-- more -->

# 依赖

两个依赖：

- spring-boot-starter-web
- spring-boot-starter-test

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

# Controller

一个简单 Controller，打印 `Hello World!`

```java
package com.ikutarian.blog.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("hello")
    public String hello() {
        return "Hello World!";
    }
}
```

# Applicatoin

```java
package com.ikutarian.blog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BlogApplication {

	public static void main(String[] args) {
		SpringApplication.run(BlogApplication.class, args);
	}

}
```

# 运行效果

启动 `BlogApplication.java` 之后，访问 `http://127.0.0.1/hello`，会返回 `Hello World!` 字符串

# 测试用例

测试一下，是否访问 `http://127.0.0.1/hello`，会返回 `Hello World!` 字符串

```java
package com.ikutarian.blog.controller;


import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(content().string("Hello World!"));

    }
}
```

# 官方 Demo

在 Spring 的官网，[有一个 Guide](https://spring.io/guides/gs/testing-web/) 是讲解如何测试 Web Layer 的，可以参考

