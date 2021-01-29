---
title: SpringApplication.run()执行流程
date: 2020-08-31 11:17:14
tags:
  - SpringBoot
  - 源码分析
categories:
  - 源码分析
---

一直不知道经常使用的 `SpringApplication.run()` 方法它里面做了什么工作，现在来一探究竟。源码分析版本是 `2.2.1.RELEASE`

<!-- more -->

利用 IDEA 查看 `run()` 方法，可以看到方法体如下

```java
/**
  * Run the Spring application, creating and refreshing a new
  * {@link ApplicationContext}.
  * @param args the application arguments (usually passed from a Java main method)
  * @return a running {@link ApplicationContext}
  */
public ConfigurableApplicationContext run(String... args) {
  StopWatch stopWatch = new StopWatch();
  stopWatch.start();
  ConfigurableApplicationContext context = null;
  FailureAnalyzers analyzers = null;
  configureHeadlessProperty();
  SpringApplicationRunListeners listeners = getRunListeners(args);
  listeners.starting();
  try {
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(
        args);
    ConfigurableEnvironment environment = prepareEnvironment(listeners,
        applicationArguments);
    Banner printedBanner = printBanner(environment);
    context = createApplicationContext();
    analyzers = new FailureAnalyzers(context);
    prepareContext(context, environment, listeners, applicationArguments,
        printedBanner);
    refreshContext(context);
    afterRefresh(context, applicationArguments);
    listeners.finished(context, null);
    stopWatch.stop();
    if (this.logStartupInfo) {
      new StartupInfoLogger(this.mainApplicationClass)
          .logStarted(getApplicationLog(), stopWatch);
    }
    return context;
  }
  catch (Throwable ex) {
    handleRunFailure(context, listeners, analyzers, ex);
    throw new IllegalStateException(ex);
  }
}
```

按照方法注释的说明

> Run the Spring application, creating and refreshing a new  {@link ApplicationContext}.

这个 `run()` 方法用来启动 Spring 应用，然后创建并刷新 `ApplicationContext` 对象

给方法体的主要部分加上注释


```java
public ConfigurableApplicationContext run(String... args) {
  StopWatch stopWatch = new StopWatch();
  stopWatch.start();
  ConfigurableApplicationContext context = null;
  FailureAnalyzers analyzers = null;
  configureHeadlessProperty();
  // 获取SpringApplicationRunListener监听器列表
  SpringApplicationRunListeners listeners = getRunListeners(args);
  // 通过监听器发出starting通知
  listeners.starting();
  try {
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(
        args);
    ConfigurableEnvironment environment = prepareEnvironment(listeners,
        applicationArguments);
    Banner printedBanner = printBanner(environment);
    context = createApplicationContext();
    analyzers = new FailureAnalyzers(context);
    prepareContext(context, environment, listeners, applicationArguments,
        printedBanner);
    refreshContext(context);
    afterRefresh(context, applicationArguments);
    listeners.finished(context, null);
    stopWatch.stop();
    if (this.logStartupInfo) {
      new StartupInfoLogger(this.mainApplicationClass)
          .logStarted(getApplicationLog(), stopWatch);
    }
    return context;
  }
  catch (Throwable ex) {
    handleRunFailure(context, listeners, analyzers, ex);
    throw new IllegalStateException(ex);
  }
}
```

# SpringApplicationRunListener 监听器

## 源码走读

`SpringApplication.run()` 方法中有两条语句是关于监听器的

```java
SpringApplicationRunListeners listeners = getRunListeners(args);
listeners.starting();
```

`SpringApplicationRunListeners` 是 `SpringApplicationRunListener` 的容器。通过构造方法将 `SpringApplicationRunListener` 集合传入，由这个这个容器类来执行监听器的 `starting`、`started`、`running` 等方法

```java
SpringApplicationRunListeners(Log log,
			Collection<? extends SpringApplicationRunListener> listeners) {
  this.log = log;
  this.listeners = new ArrayList<SpringApplicationRunListener>(listeners);
}
```

`getRunListeners(args)` 就是用来获取容器类对象


```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
  Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
  return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
      SpringApplicationRunListener.class, types, this, args));
}
```

`getRunListeners()` 方法体中的 `getSpringFactoriesInstances` 就是用来获取集合的

```java
private <T> Collection<? extends T> getSpringFactoriesInstances(Class<T> type,
    Class<?>[] parameterTypes, Object... args) {
  // 获取ClassLoader
  ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
  // 获取类名去重集合
  Set<String> names = new LinkedHashSet<String>(
      SpringFactoriesLoader.loadFactoryNames(type, classLoader));
  // 实例化
  List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
      classLoader, args, names);
  // 排序
  AnnotationAwareOrderComparator.sort(instances);
  return instances;
}
```

其中 `SpringFactoriesLoader.loadFactoryNames(type, classLoader)` 用来获取类名集合

```java
	public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		try {
      // FACTORIES_RESOURCE_LOCATION 常量值为 META-INF/spring.factories
			Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			List<String> result = new ArrayList<String>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
        // 以factoryClassName为key，寻找对应的value
				String propertyValue = properties.getProperty(factoryClassName);
        // value是以逗号分隔的字符串，所以对value进行分割
				for (String factoryName : StringUtils.commaDelimitedListToStringArray(propertyValue)) {
					result.add(factoryName.trim());
				}
			}
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

`loadFactoryNames` 会到 classpath 下找 `META-INF/spring.factories` 文件，这个文件是 properties 文件，所以调用 `PropertiesLoaderUtils` 去读取配置。因为传入的 `factoryClass` 是 `SpringApplicationRunListener.class`，所以它会以类的全名，也就是 `org.springframework.boot.SpringApplicationRunListener` 为 key，去找以逗号分隔的 value。目前 value 只有一个 

```
# 省略...

# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener

# 省略...
```

于是就把 `org.springframework.boot.context.event.EventPublishingRunListener` 返回去。

得到了类名，接着就可以利用反射进行实例化了

```java
List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
      classLoader, args, names);
```

具体实现是

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type,
    Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
    Set<String> names) {
  List<T> instances = new ArrayList<T>(names.size());
  for (String name : names) {
    try {
      Class<?> instanceClass = ClassUtils.forName(name, classLoader);
      Assert.isAssignable(type, instanceClass);
      Constructor<?> constructor = instanceClass
          .getDeclaredConstructor(parameterTypes);
      T instance = (T) BeanUtils.instantiateClass(constructor, args);
      instances.add(instance);
    }
    catch (Throwable ex) {
      throw new IllegalArgumentException(
          "Cannot instantiate " + type + " : " + name, ex);
    }
  }
  return instances;
}
```

重点是这两行

```java
Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
T instance = (T) BeanUtils.instantiateClass(constructor, args);
```

获取对应的类的构造方法，这里传入的 `parameterTypes` 是一个数组 `Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };`，所以它会找这个类的参数名为 `SpringApplication.class, String[].class` 的构造方法。最后就是利用 `T instance = (T) BeanUtils.instantiateClass(constructor, args);` 进行实例化得到对象

## 总结一下

总的来说，`SpringApplication.run()` 方法中的这两条语句

```java
SpringApplicationRunListeners listeners = getRunListeners(args);
listeners.starting();
```

它的工作就是从 `META-INF/spring.factories` 中找到 `org.springframework.boot.SpringApplicationRunListener` 对应的 key，也就是 `org.springframework.boot.context.event.EventPublishingRunListener`，然后实例化成对象。调用对象的 `starting()` 方法

## SpringApplicationRunListener 源码解析

```java
/**
 * Listener for the {@link SpringApplication} {@code run} method.
 * {@link SpringApplicationRunListener}s are loaded via the {@link SpringFactoriesLoader}
 * and should declare a public constructor that accepts a {@link SpringApplication}
 * instance and a {@code String[]} of arguments. A new
 * {@link SpringApplicationRunListener} instance will be created for each run.
 *
 * @author Phillip Webb
 * @author Dave Syer
 */
public interface SpringApplicationRunListener {
```

这是一个接口。从注释中可以看出 `SpringApplicationRunListener` 是 `SpringApplication.run()` 方法的监听器。通过 `SpringFactoriesLoader` 加载，就是上面看到的 `SpringFactoriesLoader.loadFactoryNames(type, classLoader)` 方法。并且实现类要提供一个接收 `SpringApplication` 和 `Strign[]` 参数的构造器。每次 `SpringApplication.run()` 方法执行的时候，都会创建一个新的 `SpringApplicationRunListener` 实例

`SpringApplicationRunListener` 提供了一些启动过程中的回调方法

```java
/**
 * Called immediately when the run method has first started. Can be used for very
 * early initialization.
 *
 * 当run()方法第一次启动的时候，会被立即调用。可以用于非常早期的初始化工作
 */
default void starting() {
}

/**
 * Called once the environment has been prepared, but before the
 * {@link ApplicationContext} has been created.
 * @param environment the environment
 *
 * environment已经准备完成，但是ApplicationContext还没有创建的时候
 */
default void environmentPrepared(ConfigurableEnvironment environment) {
}

/**
 * Called once the {@link ApplicationContext} has been created and prepared, but
 * before sources have been loaded.
 * @param context the application context
 */
default void contextPrepared(ConfigurableApplicationContext context) {
}

/**
 * Called once the application context has been loaded but before it has been
 * refreshed.
 * @param context the application context
 */
default void contextLoaded(ConfigurableApplicationContext context) {
}

/**
 * The context has been refreshed and the application has started but
 * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
 * ApplicationRunners} have not been called.
 * @param context the application context.
 * @since 2.0.0
 */
default void started(ConfigurableApplicationContext context) {
}

/**
 * Called immediately before the run method finishes, when the application context has
 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
 * {@link ApplicationRunner ApplicationRunners} have been called.
 * @param context the application context.
 * @since 2.0.0
 */
default void running(ConfigurableApplicationContext context) {
}

/**
 * Called when a failure occurs when running the application.
 * @param context the application context or {@code null} if a failure occurred before
 * the context was created
 * @param exception the failure
 * @since 2.0.0
 */
default void failed(ConfigurableApplicationContext context, Throwable exception) {
}
```