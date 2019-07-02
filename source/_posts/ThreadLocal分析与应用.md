---
title: ThreadLocal分析与应用
date: 2019-07-02 13:39:35
tags:
  - ThreadLocal
  - 多线程
categories:
  - 多线程
---

# ThreadLocal 是什么

`ThreadLocal` 是一个用于创建线程局部变量的类。通常情况下，我们创建的变量是可以被任何一个线程访问并修改的。而使用 `ThreadLocal` 创建的变量只能被当前线程访问，其他线程则无法访问和修改

<!-- more -->

# 用法

## 创建

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
```

## set

```java
threadLocal.set("Hello World!");
```

## get

```java
threadLocal.get();
```

## 初始值

为 `ThreadLocal` 设置默认的 `get()` 返回值，需要重写 `initialValue` 方法，下面是一段代码，我们将默认值修改成了线程的名字

```java
ThreadLocal<String> threadLocal = new ThreadLocal<String>() {

    @Override
    protected String initialValue() {
        return Thread.currentThread().getName();
    }
};
```

# 实现原理

为了更好的掌握 `ThreadLocal`，我认为了解其内部实现是很有必要的，这里我们以 `set` 方法从起始看一看 `ThreadLocal` 的实现原理

下面是 `ThreadLocal` 的 `set` 方法，大致意思为

- 首先获取当前线程
- 利用当前线程的 `ThreadLocalMap` 的对象
- 如果 `ThreadLocalMap` 对象不为空，则设置值，否则创建这个 `ThreadLocalMap` 对象并设置值

```java
/**
 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

`getMap` 方法的实现如下

```java
/**
 * Get the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param  t the current thread
 * @return the map
 */
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

`threadLocals` 就是 Thread 的属性

```java
public class Thread implements Runnable {
 
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

}
```

如果一开始调用 `set` 方法，`ThreadLocalMap` 对象未创建，则新建 `ThreadLocalMap` 对象，并设置初始值

```java
/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

所以，`ThreadLocal` 的值是放入了当前线程的 `ThreadLocalMap` 属性中，所以只能在本线程中访问，其他线程无法访问

# 使用ThreadLocal会造成内存泄露吗？

有网上讨论说ThreadLocal会导致内存泄露，原因如下：

- 首先 `ThreadLocal` 实例被线程的 `ThreadLocalMap` 实例持有，也可以看成被线程持有
- 如果应用使用了线程池，那么之前的线程实例处理完之后出于复用的目的依然存活
- 所以，`ThreadLocal` 设定的值被持有，导致内存泄露

面的逻辑是清晰的，可是 `ThreadLocal` 并不会产生内存泄露，因为 `ThreadLocalMap` 在选择 key 的时候，并不是直接选择 `ThreadLocal` 实例，而是 `ThreadLocal` 实例的弱引用

```java
static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
}
```

# 使用场景

在实际项目中，可以用来减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度，因为 `Servlet` 是单例多线程的，每个请求执行的操作都是同一个线程中。比如：可以用 `ThreadLocal` 来存每一次请求用户的信息，定义了一个类 `UserContextHolder`

```java
public class UserContextHolder{

    private static ThreadLocal<User> userContextHolder = new ThreadLocal<>();
    
    public static setUser(User user){
        userContextHolder.set(user);
    }
    
    public static User getUser(){
        return userContextHolder.get();
    }
    
    public static void remove(){
        return userContextHolder.remove();
    }
}
```

当用户每次请求进来时，在拦截器中获取用户信息调用 `UserContextHolder.setUser()` 将其放到 `userContext` 中，无论在哪我们只要调用 `UserContextHolder.getUser()`可以很轻松的获取到用户的信息，而不用在函数调用时一层一层的传递。同时在拦截器结束时调用 `UserContextHolder.remove()` 移除掉即可