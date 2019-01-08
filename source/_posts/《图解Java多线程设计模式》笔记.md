---
title: 《图解Java多线程设计模式》
date: 2018-11-07 11:24:35
tags:
  - Java
  - 多线程
categories:
  - 读书笔记
---

读书计划开始了，这是我要阅读的第一本书

<!-- more -->

# 序章1

- 什么是线程
  - 单线程和多线程、Thread 类、run 方法、start 方法
- 线程的启动
  - Thread 类、Runnable 接口
- 线程暂停
  - sleep 方法
- 线程互斥处理
  - synchronized 方法、synchronized 代码块、锁
- 线程的写作
  - 等待队列、wait 方法、notify 方法、notifyAll 方法

## 单线程

代码的执行流程只有一条线

## 多线程

代码的执行轨迹像多条线一样交织在一起

## Thread 类的 run 方法和 start 方法

start 方法

> start 方法用于启动新线程，然后由这个新线程调用 run 方法

run 方法

> 只是一个普通的 Java 方法而已

## 线程的启动

1. 利用 Thread 类的子类
2. 利用 Runnable 接口

## 线程的暂停

Thread 类的 sleep 静态方法，可以让当前线程暂停指定的时间

## synchronized 方法

一个实例中的 synchronized 方法每次只能由一个线程运行

比如一个银行类，有存钱和取钱两个 synchronized 方法

```java
public class Bank {

    public synchronized void deposit(int money) {
      // ...
    }

    public synchronized void withdraw(int money) {
      // ...
    }
}
```

如果一个线程正在执行 Bank 实例的 deposit 方法，那么其他线程就无法运行这个实例的 deposit 方法和 withdraw 方法，**因为线程拿到了实例的锁**

## synchronized 代码块

普通 synchronized 方法

```java
synchronized void method() {

}
```

类似于

```java
void method() {

    synchronized(this) {

    }
}
```

静态 synchronized 方法

```java
class Something {
    
    static synchronized void method() {

    }
}
```

类似于

```java
class Something {

    static void method() {
        synchronized(Something.class) {

        }
    }
}
```

## 线程的协作

### 等待队列

每个实例都有一个等待队列。它是在实例的 wait 方法执行后停止操作的线程的队列

## wait 方法

将线程放入等待队列。**若要执行 wait 方法，线程必须持有锁**

## notify 方法

从等待队列中取出线程。**若要执行 notify 方法，线程必须持有锁**

## notifyAll 方法

从等待队列中取出所有线程。**若要执行 notifyAll 方法，线程必须持有锁**

# Single Threaded Execution 模式

能通过这座桥的只有一个人，**因为线程拿到了实例的锁**