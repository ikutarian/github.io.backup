---
title: Logback使用总结
date: 2018-09-28 16:21:58
tags:
  - 日志
  - Logback
categories:
  - Java
---

## 什么是 Logback

Logback 是由 Log4j 创始人设计的另一个开源日志组件，Logback 为取代 Log4j 而生

## 依赖配置

打开 pom.xml，加入

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

<!-- more -->

## Hello World

```java
package org.smart4j.chapter2;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {

    private static final Logger log = LoggerFactory.getLogger(HelloWorld.class);

    public static void main(String[] args) {
        log.debug("Hello World!");
    }
}
```

输出

```
16:29:05.510 [main] DEBUG org.smart4j.chapter2.HelloWorld - Hello World!
```

## 体系结构

Logback 分为三个模块：

1. core
2. classic
3. access

core 模块是另外两个模块的基础。classic 直接实现了 slf4j 的接口，是 Log4j 的改进版。 access 与 Servlet 容器集成，提供 HTTP 访问日志的功能

## Logger、Appender 和 Encoder

Logback 建立于三个主要类之上：Logger、Appender 和 Layout。不过现在的版本，用 **Encoder 代替 Layout** 了

### Logger 的名字

每个 Logger 都有一个名字。之前的 HelloWorld 例子中，调用 LoggerFactory.getLogger(HelloWorld.class) 使用类名 org.smart4j.chapter2.HelloWorld 作为 Logger 的名字

Logger 的名字不能乱取，因为 Logback 是通过名字来决定 Logger 的继承关系的。比如名为 com.foo 的 Logger 是 com.foo.bar 之父。同理 java 是 java.util 之父，也是 java.util.Vetor 的祖先。Logback 有一个顶级 Logger，名字由 org.slf4j.Logger.ROOT_LOGGER_NAME 定义，也就是 root

理解继承关系很重要，后面的知识点都需要利用继承关系才能理解

### Logger 的级别

级别从低到高有：

- TRACE
- DEBUG
- INFO
- WARN
- ERROR

如果没有给 Logger 设置级别，那么它会从最近的祖先继承级别，更准确地说

> 从下往上，找到继承树里，第一个有设置级别的 Logger，然后从它那里继承到级别

顶级 Logger 的级别是 DEBUG

举几个例子说明一下级别的继承

例子1

|Logger 名称|分配级别|有效级别|
|:--|:--|:--|
|root|DEBUG|DEBUG|
|X|none|DEBUG|
|X.Y|none|DEBUG|
|X.Y.Z|none|DEBUG|

三个 Logger：X、X.Y 和 X.Y.Z 都没有分配级别，而 root 默认级别是 DEBUG，所以 X、X.Y 和 X.Y.Z 都从 root 继承到 DEBUG

例子2

|Logger 名称|分配级别|有效级别|
|:--|:--|:--|
|root|ERROR|ERROR|
|X|INFO|INFO|
|X.Y|DEBUG|DEBUG|
|X.Y.Z|WARN|WARN|

每个 Logger 都有设置级别，因此级别继承不发生作用

例子3

|Logger 名称|分配级别|有效级别|
|:--|:--|:--|
|root|DEBUG|DEBUG|
|X|INFO|INFO|
|X.Y|none|INFO|
|X.Y.Z|ERROR|ERROR|

只有 X.Y 没有分配级别，于是从 X 继承了 INFO 级别

例子4

|Logger 名称|分配级别|有效级别|
|:--|:--|:--|
|root|DEBUG|DEBUG|
|X|INFO|INFO|
|X.Y|none|INFO|
|X.Y.Z|none|INFO|

X.Y 和 X.Y.Z 都没有设置级别，所以都从 X 继承到 INFO

### Logger 输出日志

Logger 有日志级别。在输出日志的时候，日志级别为 p，Logger 的有效级别为 q，只有 p >= q 才能成功输出日志

### Appender 和 Encoder

日志输出的目的地，比如控制台、 文件、数据库等，在 Logback 都叫做 Appender。一个 Logger 可以把日志输出到多个目的地，也就是一个 Logger 可以关联多个 Appender

日志都有格式，比如 `%-4relative [%thread] %-5level %logger{32} - %msg%n`，在 Logback 里，由 Encoder 负责进行格式化

#### Appender 的叠加性

Logger 有继承，Logger 会关联 多个 Appender，所以 Appender 也有继承，在 Logback 里叫“叠加性（additivity）”

比如根 Logger 有一个打印到控制台的 Appender 名字叫 A_console，那么所有继承根 Logger 的子 Logger 不光会把日志输出到自己的 Appender，还会把日志输出到 A_console

所谓叠加性（additivity）

> Logger 的日志输出是否会向上发送

不过，Logger 可以设置不允许叠加。比如一个 Logger 名字叫 L，祖先 Logger 名字叫 P。那么 L 的日志输出，会发送到 L 与 P（包括 P）之间的所有 Appender。P 设置为不可叠加，那么日志输出就到此为止了，不会再向上传递

举个例子

|Logger 名称|关联的 Appender|是否叠加|输出的 Appender|说明|
|:--|:--|:--|:--|:--|
|root|A1|N|A1|root 没有叠加性|
|x|A-x1，A-x2|Y|A1，A-x1，A-x2|root 和 x 的 Appender|
|x.y|none|Y|A1，A-x1，A-x2|root 和 x 的 Appender|
|x.y.z|A-xyz|Y|1，A-x1，A-x2，A-xyz|root、x 和 x.y.z 的 Appender|
|security|A-sec|N|A-sec|不可叠加，所以日志不向上传递给 root|
|security.access|none|Y|A-sec|可叠加，于是日志向上传递给 security。但是 security 不可叠加，于是日志到此为止。于是只有 A-sec|


#### Appender 叠加性的陷阱

```xml
<configuration>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.smart4j.util">
        <appender-ref ref="STDOUT" />
    </logger>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

打印一条日志

```java
package org.smart4j.util;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Foo {

    private static final Logger log = LoggerFactory.getLogger(Foo.class);

    public static void main(String[] args) {
        log.debug("a line log");
    }
}
```

控制台输出

```
16:36:41.066 [main] DEBUG org.smart4j.util.Foo - a line log
16:36:41.066 [main] DEBUG org.smart4j.util.Foo - a line log
```

可以看到日志重复输出了。这是因为 root 和 com.smart4j.util 两个 Logger 都关联了 `STDOUT` 的 Appender，又因为 Appender 的叠加性，com.smart4j.util 的日志会向上传递给 root。所以，就会输出两条日志了

不过，也可以利用这个陷阱，让所有 Logger 的日志输出到控制台，某些 Logger 的日志输出到其他地方，比如文件

```xml
<configuration>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>myApp.log</file>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.smart4j.util">
        <appender-ref ref="FILE" />
    </logger>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

如果配置文件是这样的，所有的日志都会输出到控制台，`com.smart4j.util` 包下到日志还会输出到文件

如果 `com.smart4j.util` 包下到日志只输出到文件，只需要设置 Appender 为不可叠加即可

```xml
<logger name="com.smart4j.util" additivity="false">
    <appender-ref ref="FILE" />
</logger>
```

## 阶段总结

到目前为止，学到到重点内容是

- 三剑客：Logger、Appender、Encoder
- Logger 的继承
- 日志级别的继承
- Appender 的叠加性

## 配置文件

### 模板

在新建 `logback.xml` 文件

```xml
<configuration>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

### 配置文件基本结构

以 `<configuration>` 开头，后面有零个或多个 `<appender>` 元素，有零个或多个 `<logger>` 元素，有最多一个 `<root>` 元素

### 开启debug模式

```xml
<configuration debug="true">

    ...

</configuration>
```

### 热加载

```xml
<configuration scan="true" scanPeriod="30 seconds">

    ...

</configuration>
```

### 语法

#### logger 语法

```xml
<logger name="" level="" additivity="">
    <appender-ref ref="">
</logger>
```

`name` 必填，`level` 和 `additivity` 可选。一个 Logger 可以关联 0 个或多个 Appender，所以 `appender-ref` 可以有 0 个或多个

#### root 语法

```xml
<root level="">
    <appender-ref ref="">
</root>
```

`appender-ref` 可以有 0 个或多个

#### 例子

一个名为 `STDOUT` 的 Appender，root 日志级别为 DEBUG，关联了 `STDOUT`。日志级别大于等于 DEBUG 的所有日志都会输出到 `STDOUT`

```xml
<configuration>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

但是，唯独 `com.smart4j.util` 包下的日志，只输出大于等于 INFO 级别的日志

```xml
<configuration>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.smart4j.util" level="info" />

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

名为 `com.smart4j.util` 的 Logger 没有关联 Appender，而且也没有关闭叠加性，所以日志会向上传递到 `STDOUT`

还可为多个 Logger 设置级别，比如 `com.smart4j.util.Foo` 类的级别设置为 DEBUG

```xml
<configuration>
    
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="com.smart4j.util" level="info" />

    <logger name="com.smart4j.util.Foo" level="debug" />

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

#### appender 语法

```xml
<appender name="" class="">
    <encoder>
    </encoder>
    
    <filter>
    </filter>
</appender>
```

`name` 必填，指定名称。`class` 必填，指定类名。零个或多个的 `encoder` 和 `filter`

`encoder` 和 `filter` 都有 `class` 属性。常用的 `encoder` 是 `PatternLayoutEncoder`，所以 `class` 属性可以忽略不写，比如下面的例子，

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

## Appender

常用的 Appender 有两种：控制台、文件

控制台的 Appender 常用的只有一个：

- ConsoleAppender

文件的 Appender 有两个

- FileAppender
- RollingFileAppender

### ConsoleAppender

ConsoleAppender 把日志输出到到控制台

#### 属性

|属性名|类型|描述|
|:--|:--|:--|
|encoder|Encoder|指定日志格式|
|target|String|可选值：System.out 或 System.err。默认是 System.out|

#### 一个例子

要把日志输出到控制台，一般都这么写

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

target　默认使用的是 System.out，控制台输出的文字是白色的

{% asset_img QQ20180930-140946@2x.png %}

改成 System.err 的话

```xml
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <target>System.err</target>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

就变成红色的了

{% asset_img QQ20180930-141335@2x.png %}

### FileAppender

把日志输出到文件

#### 属性

|属性名|类型|描述|
|:--|:--|:--|
|file|String|日志文件名|
|encoder|Encoder|指定日志格式|
|append|boolean|默认为 true。true：日志追加到文件尾部。false：清空现存文件|

这个例子，把日志写到 log 文件夹下的 testFile.log 文件中，并且每次写入日志都追加到文件尾部

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>log/testFile.log</file>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

append 属性不是很常用，因为每次写入都会把 testFile.log 文件清空再写入日志

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>log/testFile.log</file>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
    <append>false</append>
</appender>
```

如果有这么一个需求：程序每次启动时，都以当前时间为日志的文件名，比如 log-2018-09-30_15:14:49.log，然后程序之后的所有日志都写入到 log-2018-09-30_15:14:49.log 中。下一次程序再次启动，又会创建一个以当前时间为文件名的日志文件，再把日志写进去。这样，每次程序启动到程序结束，所有的日志输出都在一个独立的文件里面

配置文件可以这么写

```xml
<configuration>

    <timestamp key="bySecond" datePattern="yyyy-MM-dd_HH:mm:ss" />

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>log/log-${bySecond}.log</file>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="FILE" />
    </root>

</configuration>
```

### RollingFileAppender

RollingFileAppender 和 FileAppender 一样，都是把日志输出到文件。只不过 RollingFileAppender 可以在满足对应条件时，做出一些行为

RollingFileAppender 要与两个子标签配合才行：

- rollingPolicy
- triggeringPolicy

不过有的 RollingPolicy 也实现了 TriggeringPolicy 接口，所以有时候只需要设置 rollingPolicy 标签就行了

#### 属性

|属性名|类型|描述|
|:--|:--|:--|
|file|String|日志文件名|
|encoder|Encoder|指定日志格式|
|append|boolean|默认为 true。true：日志追加到文件尾部。false：清空现存文件|
|rollingPolicy|RollingPolicy|滚动策略，决定 RollingFileAppender 的行为|
|triggeringPolicy|TriggeringPolicy|告知 RollingFileAppender 何时滚动|

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>testFile.log</file>
    <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
    <triggeringPolicy class="">
        ...
    </triggeringPolicy>
    <rollingPolicy class="">
        ...
    </rollingPolicy>
</appender>
```

triggeringPolicy 告知 RollingFileAppender 何时激活滚动。当发生滚动时，rollingPolicy 决定 RollingFileAppender 的行为（比如文件压缩、移动和重命名等）

常用的 triggeringPolicy 有：

- SizeBasedTriggeringPolicy

常用的 RollingPolicy 有：

- FixedWindowRollingPolicy
- TimeBasedRollingPolicy

#### FixedWindowRollingPolicy

当日志文件滚动时，FixedWindowRollingPolicy 可以根据固定窗口（Fixed Window）算法重命名文件

属性有这些

|属性名|类型|描述|
|:--|:--|:--|
|minIndex|int|窗口索引的最小值，最小值为 1|
|maxIndex|int|窗口索引的最大值，最大值为 20|
|fileNamePattern|String|文件名模式，支持 zip、gz 压缩，%i 关键词不可省略|

比如，指定最小索引值为 1，最大索引值为 3，文件名模式为 mylog%i.log

|滚动文件数量|日志输出文件|归档文件|描述|
|:--|:--|:--|:--|
|0|mylog.log|无|还没发生滚动，日志会写入到 mylog.log 中|
|1|mylog.log|mylog1.log|第一次滚动。mylog.log 被重命名为 mylog1.log。创建新的 mylog.log 文件继续写入日志|
|2|mylog.log|mylog1.log、mylog2.log|第二次滚动。mylog1.log 被重命名为 mylog2.log。mylog.log 被重命名为 mylog1.log。创建新的 mylog.log 文件继续写入日志|
|3|mylog.log|mylog1.log、mylog2.log、mylog3.log|第三次滚动。mylog2.log 被重命名为 mylog3.log。mylog1.log 被重命名为 mylog2.log。mylog.log 被重命名为 mylog1.log。创建新的 mylog.log 文件继续写入日志|
|4|mylog.log|mylog1.log、mylog2.log、mylog3.log|会先把 mylog3.log 删除，然后再按照上面的步骤重命名文件，然后再生成一个新的 mylog.log 文件继续写入日志|

例子：日志文件最大为 5MB，分成 3 个文件

```xml
<configuration>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/myLog.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>log/myLog%i.log</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>3</maxIndex>
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <maxFileSize>5MB</maxFileSize>
        </triggeringPolicy>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="FILE" />
    </root>

</configuration>
```

#### TimeBasedRollingPolicy

根据时间滚动日志文件，比如每天生成一个日志文件。TimeBasedRollingPolicy 同时实现了 RollingPolicy 接口和 TriggeringPolicy 接口

属性有两个

|属性名|类型|描述|
|:--|:--|:--|
|fileNamePattern|String|日志文件的名字格式，由“文件名”+“日期格式”组成。如果没有制定日期格式，默认时 yyyy-MM-dd|
|maxHistory|int|日志文件保存最大时长，超过就删除旧文件|

`fileNamePattern` 的日期格式由 `%d{}` 表示，`{}` 里面放 java.text.SimpleDateFormat 指定的日期时间模式，比如 ` yyyy-MM-dd`。根据不同的时间格式，可以表似乎每天滚动、每月滚动、每周滚动等

|fileNamePattern|滚动计划|
|:--|:--|
|%d|每天滚动|
|%d{yyyy-MM}|每月滚动|
|%d{yyyy-ww}|每周滚动|
|%d{yyyy-MM-dd_HH}|每小时滚动|
|%d{yyyy-MM-dd_HH-mm}|每分钟滚动|

`maxHistory` 的值需要根据 `fileNamePattern` 匹配，如果 `fileNamePattern` 是每天滚动，`maxHistory` 设为 6 就表示最多保留 6 天日志记录，以此类推

例子：每天滚动，最长保留30天

```xml
<configuration>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/myLog.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/myLog_%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="FILE" />
    </root>

</configuration>
```

logback 还提供了根据时间和日志文件大小的滚动策略（SizeAndTimeBasedFNATP），比如下面的例子：每天滚动、最多保存 30 天、文件大小最多 100MB

注意 `fileNamePattern` 需要两个标识符：`%d` 和 `%i`

```xml
<configuration>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/myLog.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/myLog_%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy 
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="FILE" />
    </root>

</configuration>
```

## Encoder

负责指定日志的输出格式。常用的 Encoder 只有一个：

- PatternLayoutEncoder

Encoder 的知识不多，只需要掌握日志格式怎么写就行，它时由各种各样的日志转换符、格式修饰符组成的。也就是下面这段配置中的 pattern 部分

```xml
<encoder>
    <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
```

### 日志转换符

现在就来看一下日志格式转换符要怎么写，常用的有几个

* logger - 输出日志的 Logger 的名称
* class - 调用 Logback 的类的名称
* date - 日志输出时的时间
* file - 调用 Logback 的类所在的 Java 文件名
* line - 调用 Logback 的代码所在的行号
* method - 调用 Logback 的方法名
* thread - 调用 Logback 的线程
* message - 日志内容
* n - 日志换行符
* level - 日志的级别

Logback 规定，日志格式转换符前面都要带一个 `%` 符号，表示这是一个日志格式转换符

下面分别说明，为了方便说明，写了一个简单的 Java 类

```java
package com.ikutarian.logback;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {

    private static final Logger log = LoggerFactory.getLogger(HelloWorld.class);

    public static void main(String[] args) {
        log.debug("Hello World!");
    }
}
```

#### logger - 输出日志的 Logger 的名称

在 Logback 中，logger 有三种写法：

- c
- lo
- logger

可以指定 logger 名称的长度，在后面带一个 `{length}` 字符串表示长度即可。不过**注意一下**：

1. 最右边的 logger 名（也就是最后一个点号之后的字符）永远不被省略，即使它的长度超过了 `length` 指定的长度

2. logger 名里的其他片段可以被缩短为至少 1 个字符，但永远不会消失

下面对一个名称为 com.ikutarian.logback.HelloWorld 的 logger 使用日志转换符之后得到的结果进行举例

|日志格式转换符|转换后的结果|解释|
|:--|:--|:--|
|%logger|com.ikutarian.logback.HelloWorld|没有指定长度，直接输出全名|
|%logger{0}|HelloWorld|设为0表示只输出 logger 名里最右边的点号之后的字符串|
|%logger{5}|c.i.l.HelloWorld|只保留5个字符，也就是 `c.i.l.` 这5个|
|%logger{10}|c.i.l.HelloWorld|只保留10个字符，`c.i.l.` 占5个，后面的 `HelloWorld` 占10个，但是不会被省略|
|%logger{15}|c.i.l.HelloWorld|只保留15个字符，`c.i.l.` 占5个，后面的 `HelloWorld` 占10个，但是不会被省略|
|%logger{20}|c.i.l.HelloWorld|只保留20个字符，`c.i.l.` 占5个，后面的 `HelloWorld` 占10个，但是不会被省略|
|%logger{25}|c.i.logback.HelloWorld|只保留25个字符，`c.i.logback.` 占12个，后面的 `HelloWorld` 占10个，但是不会被省略|
|%logger{30}|c.ikutarian.logback.HelloWorld|只保留30个字符，`c.ikutarian.logback.` 占20个，后面的 `HelloWorld` 占10个，但是不会被省略|
|%logger{35}|c.ikutarian.logback.HelloWorld|`c.ikutarian.logback.HelloWorld` 本身是32个字符。只保留35个字符，所以可以显示完全|

#### class - 调用 Logback 的类的名称

有两种写法

- C
- class

和 `logger` 一样，也可以带上 `{length}` 参数，用法也是一样的

#### date - 日志输出时的时间

两种写法：

- d
- date

可带上时间格式 `{pattern}`。pattern 的写法和 java.text.SimpleDateFormat 的格式兼容，所以就按照 java.text.SimpleDateFormat 的格式来写就行，比如 `%date{yyyy-MM-dd HH:mm:ss}`

#### file - 调用 Logback 的类所在的 Java 文件名

两种写法：

- F
- file

#### line - 调用 Logback 的代码所在的行号

两种写法：

- L
- line

#### method - 调用 Logback 的方法名

两种写法：

- M
- method

#### thread - 调用 Logback 的线程

两种写法：

- t
- thread

#### message - 日志内容

三种写法：

- m
- msg
- message

#### n - 日志换行符

只有一种写法：

- n

#### level - 日志的级别

三种写法：

- p
- le
- level

#### 一个综合例子

```
<pattern>%date{yyyy-MM-dd HH:mm:ss} %file %line %method [%thread] %level %logger - %message%n</pattern>
```

输出

```
2018-10-21 14:41:40 HelloWorld.java 11 main [main] DEBUG com.ikutarian.logback.HelloWorld - Hello World!
```

### 格式修饰符

默认情况下，相关信息会按原格式输出。但是，在格式修饰符的帮助下，就可以为每个字段指定最小、最大宽度，以及对齐方式。格式修饰符要放在 `%` 与转换符之间

常用的有几个：

- 左对齐
- 最小宽度

#### 左对齐

符号时减号：`-`

#### 最小宽度

如果字符小于最小宽度，就左填充（其实就是右对齐），填充字符是空格。如果字符大于最小宽度，就扩张字符宽度，字符永远不会被截断

#### 例子

```java
public class HelloWorld {

    private static final Logger log = LoggerFactory.getLogger(HelloWorld.class);

    public static void main(String[] args) {
        log.debug("12345");
    }
}
```

日志转换符只使用 `%message`，程序运行时会输出一条日志 `12345`。下面来试一试各种日志修饰符的效果

|日志修饰符|结果|说明|
|:--|:--|:--|
|%10message|&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;12345|字符长度为5，最小宽度是10，所以会左填充5个空格|
|%-10message|12345&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|最小宽度是10，左对齐，右边填充5个空格|
|%2message|12345|最小宽度为2，字符宽度为5，宽度自动扩张，不会被截断|

## Filter

只有三个返回值：

- ACCEPT
- DENY
- NEUTRAL

如果返回 DENY，那么记录事件立即被抛弃，不再经过剩余过滤器。 如果返回 NEUTRAL，那么有序列表里的下一个过滤器会接着处理记录事件。如果返回 ACCEPT，那么记录事件被立即处理，不再经过剩余过滤器

过滤器可以被添加到 Appender 中。为 Appender 添加一个或多个过滤器后，你可以用任意条件对事件行过滤

Filter 就一个常用的：

- LevelFilter

LevelFilter 根据记录级别对日志进行过滤。如果日志的级别等于配置的级别，过滤器会根据 onMatch 和 onMismatch 属性接受或拒绝事件

比如下面的例子

这是 Java 代码

```java
public class HelloWorld {

    private static final Logger log = LoggerFactory.getLogger(HelloWorld.class);

    public static void main(String[] args) {
        log.debug("我是 debug 级别的日志");
        log.info("我是 info 级别的日志");
        log.warn("我是 warn 级别的日志");
        log.error("我是 error 级别的日志");
    }
}
```

这是 logback.xml 配置

```xml
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%message%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

因为 root 指定的 level 是 debug，高于 degbug 的所有日志都会被输出。所以运行程序，会输出

```
我是 debug 级别的日志
我是 info 级别的日志
我是 warn 级别的日志
我是 error 级别的日志
```

如果给 STDOUT 加上级别过滤器，只接受 info 级别的日志

```xml
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%message%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

现在，运行程序，只会输出

```
我是 info 级别的日志
```

## 常用的logback.xml模板

1. 日志最低级别是 debug
2. 日志有两个输出目的地：控制台、输出到当前路径下的 log 文件夹中 myApp.log 文件内
3. 日志的编码都是 UTF-8
4. 日志文件每天滚动，最长保存30天，每个日志文件最大10MB

```xml
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/myLog.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>log/myLog_%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILE"/>
    </root>

</configuration>
```

对于 info 和 error 级别的日志，需要分开写入日志文件。info 的日志放入 info 文件夹，error 的日志放入 error 文件夹。对于这种需求，可以使用级别过滤器

```xml
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE_INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/info/myLog.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>log/info/myLog_%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="FILE_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>log/error/myLog.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>log/error/myLog_%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <root level="debug">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="FILE_INFO"/>
        <appender-ref ref="FILE_ERROR"/>
    </root>

</configuration>
```