---
title: Jackson对Long类型的处理导致精度丢失的问题
date: 2019-09-19 10:54:20
tags:  
  - SpringBoot
  - Jackson
  - Long
  - Integer
  - 精度
  - JavaScript
categories:
  - SpringBoot
---

表的某一个字段的类型是 `BIGINT`，对应的 Java 类的属性的类型就是 `Long`。当这个字段的值由后端返回给前端网页时，发现了精度丢失的问题。比如后端返回的值是 `588085469986509185`，到了前端是 `588085469986509200`，后面的几位数变成了 0，精度丢失了

<!-- more -->

# 原因

JavaScript 中数字的精度是有限的，`BIGINT` 类型的的数字超出了 JavaScript 的处理范围。JavaScript 遵循 IEEE 754 规范，采用双精度存储（double precision），占用 64 bit

各位的含义如下：

- 1位（s） 用来表示符号位
- 11位（e） 用来表示指数
- 52位（f） 表示尾数

尾数位最大是 52 位，因此 JS 中能精准表示的最大整数是 `Math.pow(2, 53)`，十进制即 `9007199254740992`。而 `BIGINT` 类型的有效位数是63位（扣除一位符号位），其最大值为：`Math.pow(2, 63)`。任何大于 `9007199254740992` 的就可能会丢失精度

# 解决方法

解决办法就是让 JavaScript 把数字当成字符串进行处理。对 JavaScript 来说，不进行运算，数字和字符串处理起来没有什么区别

在 Springboot 中处理方法基本上有以下几种：

## 1. 配置参数 write_numbers_as_strings

Jackson 有个配置参数 `WRITE_NUMBERS_AS_STRINGS`，可以强制将所有数字全部转成字符串输出。其功能介绍为：

> Feature that forces all Java numbers to be written as JSON strings.

使用方法很简单，只需要配置参数即可

```yml
spring:
  jackson:
    generator:
      write_numbers_as_strings: true
```

这种方式的优点是使用方便，不需要调整代码；缺点是颗粒度太大，所有的数字都被转成字符串输出了，包括按照 `timestamp` 格式输出的时间也是如此

## 2. 给 Java 类的属性单独加注解

```java
@JsonSerialize(using=ToStringSerializer.class)
private Long bankcardHash;
```

指定了 `ToStringSerializer` 进行序列化，将数字编码成字符串格式。这种方式的优点是颗粒度可以很精细；缺点同样是太精细，如果需要调整的字段比较多会比较麻烦

## 3. 自定义ObjectMapper

最后想到可以单独根据类型进行设置，只对 `Long` 型数据进行处理，转换成字符串，而对其他类型的数字不做处理。Jackson 提供了这种支持。方法是对 `ObjectMapper` 进行定制。根据 SpringBoot 的[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-spring-mvc.html#howto-customize-the-jackson-objectmapper)，找到一种相对简单的方法，只对 `ObjectMapper` 进行定制，而不是完全从头定制，方法如下：

```java
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import org.springframework.boot.autoconfigure.jackson.Jackson2ObjectMapperBuilderCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return jacksonObjectMapperBuilder -> {
            // long -> String
            jacksonObjectMapperBuilder.serializerByType(Long.TYPE, ToStringSerializer.instance);
            // Long -> String
            jacksonObjectMapperBuilder.serializerByType(Long.class, ToStringSerializer.instance);
        };
    }
}
```

如果是 Json 工具类，可以这么设置

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;

public class JsonUtil {

    private static ObjectMapper mapper = new ObjectMapper();

    static {
        SimpleModule simpleModule = new SimpleModule();
        // long -> String
        simpleModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
        // Long -> String
        simpleModule.addSerializer(Long.class, ToStringSerializer.instance);
        mapper.registerModule(simpleModule);
    }    
}
```

# 总结

总共有 3 种方式

1. 配置参数 `write_numbers_as_strings`
2. 给 Java 类的属性单独加注解
3. 自定义 `ObjectMapper`