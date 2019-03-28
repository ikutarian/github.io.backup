---
title: Spring知识总结
date: 2019-03-11 09:55:26
tags:
  - Spring
  - 总结
categories:
  - Spring
---

学习《Java工程师》的视频时，对目前掌握的 Spring 的知识做一个总结

<!-- more -->

# Spring 入门

## 什么是 Spring

Spring 是一个开源框架

Spring 为简化企业级应用开发而生。使用 Spring 可以用简单的 JavaBean 实现以前只有 EJB 才能实现的功能

百度百科的介绍：

> Spring 是一个开放源代码的设计层面框架，它解决的是业务逻辑层和其他各层的松耦合问题，因此它将面向接口的编程思想贯穿整个应用

Spring 是 JavaSE/EE 的一站式框架

## Spring 的优点

1. 方便解耦，简化开发

Spring 就是一个大工厂，可以将所有的对象创建和依赖关系维护交给 Spring 管理

2. AOP 编程的支持

Spring 提供面向切面编程，可以方便地实现对程序进行权限拦截、运行监控等功能

3. 声明式事务的支持

只需要通过配置就可以完成对事务的管理，而无需手动编程

4. 方便程序的测试

Spring 对 JUnit4 支持，可以通过注解方便地测试 Spring 程序

5. 方便集成各种优秀框架

Spring 不排斥各种优秀的开源框架，其内部提供了对各种优秀框架的直接支持

## Spring 的模块

{% asset_img spring-overview.png %}

## 源码下载地址

https://repo.spring.io/libs-release-local/org/springframework/spring/

## Spring 的 IoC 的底层实现原理

### 传统方式的开发

在 Java Web 项目里，需要在 Controller 里创建 Serviec 层的类的实例

在很早以前，可能连个接口都没有，直接就在 Controller 层创建了一个 Service 层的类的实例

```java
UserService us = new UserService();
```

这种方式并不好，因为它没有面向接口编程

### 面向接口编程

面向接口编程就需要一个接口和实现类，代码就变成了这样

```java
UserService us = new UserServiceImpl();
```

但是这种方式也不是特别的好，因为我们在 Controller 层直接创建了接口的实现类。这样 Controller 就与 Service 层产生了耦合

一个好的程序设计，应该满足 OCP 原则。所谓 OCP 原则：

> 即 open-close 原则。对程序的扩展是 open，对修改程序的代码是 close。尽量做到不修改程序的源码，然后实现对程序的扩展

上面的代码，如果要换一个 `UserService` 的实现类，就需要手动地去修改代码，这样就是产生了耦合。如果要让它不产生耦合，可以使用“工厂模式”

### 工厂模式

让工厂去创建 `UserService` 的实现类，改代码也只需要改动工厂的代码

工厂类

```java
class BeanFactory {

    public static UserService getUserService() {
        return new UserServiceImpl();
    }
}
```

Controller 层的代码

```java
UserService us = BeanFactory.getUserService();
```

现在接口和实现类就没有了耦合性了，但是接口现在依赖工厂类，接口和工厂类产生了耦合。如果要切换实现类的话，就需要修改工厂类的代码

有没有一种办法，使我们不需要去修改任何的源代码呢？这时候就可以使用“工厂 + 反射 + 配置文件”的方式

### 工厂 + 反射 + 配置文件

一个配置文件

```xml
<bean id="us" class="com.ikutarian.UserServiceImpl" />
```

工厂类

```java
class BeanFactory {

    public static Object getBean(String id) {
        // 读取配置文件
        // 根据 id 找到类
        // 利用反射实例化这个类
        // 返回
    }
}
```

“工厂 + 反射 + 配置文件” 就是 Spring 实现 IoC 的底层原理

## 理解 IoC 控制反转和 DI 依赖注入

### IoC

Inverse of Control 的缩写

控制反转的概念，就是将原本在程序中手动创建 `UserService` 对象的工作，交由 Spring 框架来完成。简单说就是将 `UserService` 对象的控制权被反转到了 Spring 框架

### DI

Dependency Injection 的缩写

DI 是依赖 IoC 的，先有了 IoC 才有 DI

依赖注入的概念，就是在 Spring 创建这个对象的过程中，将这个对象所依赖的属性注入进去。比如

```xml
<bean id="us" class="com.ikutarian.UserServiceImpl">
    <property name="name" value="张三">
</bean>
```

不需要去改动 Java 的代码，只需要改配置文件就行

# Bean 的管理

## 容器

在基于 Spring 的应用中，Bean 生存于 Spring 容器（container）中。容器负责创建、装配，配置并管理 Bean 的整个生命周期，从生存到死亡

容器有很多个，可以归为两种类型：

1. BeanFactory
2. ApplicationContext

`BeanFactory` 是最简单的容器，`ApplicationContext` 的功能更加强大

## Bean 的实例化方式

我们把 Bean 交给 Spring 容器管理。Spring 容器有三种方式来实例化 Bean

1. 类的构造器（默认使用无参构造器）
2. 静态工厂方法（简单工厂模式）
3. 实例工厂方法（工厂方法模式）

默认情况下用第一种，如果类的构造特别复杂，就会用第二种或者第三种

### 类的构造器（默认使用无参构造器）

一个 Bean

```java
public class Bean {

    public Bean() {
        System.out.println("Bean被实例化了");
    }
}
```

配置文件 applicationContext.xml

```xml
<bean id="bean" class="com.ikutarian.demo.Bean"/>
```

测试类

```java
public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        applicationContext.close();
    }
}
```

运行测试类之后可以看到控制台输出

```
Bean被实例化了
```

### 静态工厂方法（简单工厂模式）

一个 Bean

```java
public class Bean {

    public Bean() {
        System.out.println("Bean被实例化了");
    }
}
```

一个工厂，工厂里有一个静态方法

```java
public class BeanFactory {

    public static Bean createBean() {
        return new Bean();
    }
}
```

配置文件 applicationContext.xml

```xml
<bean id="bean" class="com.ikutarian.demo.BeanFactory" factory-method="createBean"/>
```

测试类

```java
public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        applicationContext.close();
    }
}
```

运行测试类之后可以看到控制台输出

```
Bean被实例化了
```

### 实例工厂方法（工厂方法模式）

一个 Bean

```java
public class Bean {

    public Bean() {
        System.out.println("Bean被实例化了");
    }
}
```

一个工厂，工厂里有一个方法

```java
public class BeanFactory {

    public Bean createBean() {
        return new Bean();
    }
}
```

配置文件 applicationContext.xml

```xml
<bean id="factory" class="com.ikutarian.demo.BeanFactory"/>

<bean id="bean" factory-bean="factory" factory-method="createBean"/>
```

测试类

```java
public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        applicationContext.close();
    }
}
```

运行测试类之后可以看到控制台输出

```
Bean被实例化了
```

## Bean 的配置

常用的配置有：

- id、name
- class
- scope

### id、name

id 不能有特殊字符，name 可以有特殊字符

### class

指定 Bean 的类全名

### scope

scope 有 4 种，默认是 singleton

- singleton：在 Spring 容器中，Bean 以单例的形式存在
- prototype：每次调用 `getBean()` 时都会返回一个新的实例
- request：每次 HTTP 请求都会创建一个新的 Bean。该作用域仅存在于 `WebApplicationContext` 环境
- session：同一个 HTTP Session 共享一个 Bean，不同的 HTTP Session 使用不同的 Bean。该作用域仅存在于 `WebApplicationContext` 环境

## Bean 的生命周期

### 两个方法

Spring 初始化 Bean 或者销毁 Bean 时，有时候需要做一些处理工作，因此 Spring 可以在创建和销毁 Bean 时调用两个生命周期方法：

- init-method：当 Bean 被载入到容器时调用
- destroy-method：当 Bean 从容器中删除的时候调用（**仅当 scope 为 singleton 有效**）

一个 Bean

```java
public class Bean {

    public Bean() {
        System.out.println("Bean 被实例化");
    }

    public void setUp() {
        System.out.println("Bean 被初始化");
    }

    public void teardown() {
        System.out.println("Bean 被销毁");
    }
}
```

配置文件 applicationContext.xml

```xml
<bean class="com.ikutarian.demo.Bean" init-method="setUp" destroy-method="teardown"/>
```

测试类

```java
public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        applicationContext.close();
    }
}
```

运行之后可以看到控制台输出

```
Bean 被实例化
Bean 被初始化
Bean 被销毁
```

如果把 `scope` 改为 `prototype`

```xml
<bean id="bean" class="com.ikutarian.demo.Bean" 
    init-method="setUp" 
    destroy-method="teardown"
    scope="prototype"/>
```

测试类

```java
public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        applicationContext.getBean("bean");
        applicationContext.close();
    }
}
```

运行之后可以看到控制台输出

```
Bean 被实例化
Bean 被初始化
```

这说明 `scope` 改为 `prototype` 时，`destroy-method` 指定的方法无法被调用

### 完整的生命周期

在传统的 Java 应用中，bean 的生命周期很简单。使用 Java 关键字 new 进行 bean 实例化，然后该 bean 就可以使用了。一旦该 bean 不再被使用，则由 Java 自动进行垃圾回收

Spring 容器中的 bean 的生命周期就显得相对复杂多了。正确理解 Spring bean 的生命周期非常重要，因为你或许要利用 Spring 提供的扩展点来自定义 bean 的创建过程

{% asset_img bean_lifecycle.png %}

1. instantiate bean 对象实例化
2. populate properties 设置属性
3. 如果 Bean 实现 BeanNameAware，会执行 setBeanName 方法（目的是让 Bean 了解它在工厂中的名称是什么）
4. 如果 Bean 实现 BeanFactoryAware 或者 ApplicationContextAware，会调用 setBeanFactory 或者 setApplicationContext（目的是让 Bean 了解工厂的信息）
5. 如果存在类实现 BeanPostProcessor（后处理 Bean），执行 postProcessBeforeInitializatoin 方法
6. 如果 Bean 实现 InitializingBean，执行 afterPropertiesSet 方法（属性设置之后）
7. 调用 <bean init-method="init"> 指定的初始化方法 init
8. 如果存在类实现 BeanPostProcessor（后处理 Bean），执行 postProcessAfterInitialization 方法
9. 执行 Bean 本身的方法，完成业务处理
10. 如果 Bean 实现 DisposableBean，执行 destroy 方法
11. 调用 <bean destroy-method="customDestroy"> 指定的销毁方法 customDestroy

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class User implements BeanNameAware, ApplicationContextAware, InitializingBean, DisposableBean {

    private String name;

    public User() {
        System.out.println("1. instantiate bean 对象实例化");
    }

    public void setName(String name) {
        System.out.println("2. populate properties 设置属性");
        this.name = name;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("3. 如果 Bean 实现 BeanNameAware，会执行 setBeanName 方法");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("4. 如果 Bean 实现 BeanFactoryAware 或者 ApplicationContextAware，会调用 setBeanFactory 或者 setApplicationContext");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("6. 如果 Bean 实现 InitializingBean，执行 afterPropertiesSet 方法（属性设置之后）");
    }

    public void init() {
        System.out.println("7. 调用 <bean init-method=\"init\"> 指定的初始化方法 init");
    }

    public void run() {
        System.out.println("9. 执行 Bean 本身的方法，完成业务处理");
    }

    public void customDestroy() {
        System.out.println("11. 调用 <bean destroy-method=\"customDestroy\"> 指定的销毁方法 customDestroy");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("10. 如果 Bean 实现 DisposableBean，执行 destroy 方法");
    }
}
```

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("5. 如果存在类实现 BeanPostProcessor（后处理 Bean），执行 postProcessBeforeInitialization 方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("8. 如果存在类实现 BeanPostProcessor（后处理 Bean），执行 postProcessAfterInitialization 方法");
        return bean;
    }
}
```

```xml
    <bean id="user" class="com.ikutarian.demo.User" init-method="init" destroy-method="customDestroy">
        <property name="name" value="张三"/>
    </bean>

    <bean class="com.ikutarian.demo.MyBeanPostProcessor"/>
```

```java
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = (User) applicationContext.getBean("user");
        user.run();
        applicationContext.close();

    }
}
```