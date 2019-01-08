---
title: 一个System.out与Throwable.printStackTrace输出信息顺序错乱的问题
date: 2018-09-14 15:17:19
tags:
  - 异常处理
categories:
  - Java
---

昨天遇到了一个 `System.out` 与 `Throwable.printStackTrace()` 输出信息顺序错乱的问题。

## 现场还原

首先我调用 `System.out` 输出普通字符串

```
Time: 0.2
------------------------
!!!FAILURES!!!
Test Results:
 Failures: 1 Errors: 1
There was 1 error:
0) test.MathTest.ok
There was 1 failure:
0) test.MathTest.add expected:"1" but was:"2"
```

然后再调用 `Throwable.printStackTrace()` 输出异常信息

```
java.lang.NoSuchMethodException: test.MathTest.ok()
	at java.lang.Class.getMethod(Class.java:1786)
	at com.okada.junit.TestCase.runTest(TestCase.java:51)
	at com.okada.junit.TestCase.run(TestCase.java:31)
	at com.okada.junit.TestSuite.run(TestSuite.java:23)
	at com.okada.junit.TestRunner.run(TestRunner.java:44)
	at com.okada.junit.TestRunner.main(TestRunner.java:37)
	at test.MathTest.main(MathTest.java:31)
```

<!-- more -->

正常来说应该能得到如下输出内容

```
Time: 0.2
------------------------
!!!FAILURES!!!
Test Results:
 Failures: 1 Errors: 1
There was 1 error:
0) test.MathTest.ok
There was 1 failure:
0) test.MathTest.add expected:"1" but was:"2"
java.lang.NoSuchMethodException: test.MathTest.ok()
	at java.lang.Class.getMethod(Class.java:1786)
	at com.okada.junit.TestCase.runTest(TestCase.java:51)
	at com.okada.junit.TestCase.run(TestCase.java:31)
	at com.okada.junit.TestSuite.run(TestSuite.java:23)
	at com.okada.junit.TestRunner.run(TestRunner.java:44)
	at com.okada.junit.TestRunner.main(TestRunner.java:37)
	at test.MathTest.main(MathTest.java:31)
```

但是有时候多运行几次代码会得到错乱的信息

```
Time: 0.2
	at java.lang.Class.getMethod(Class.java:1786)
------------------------
	at com.okada.junit.TestCase.runTest(TestCase.java:51)
!!!FAILURES!!!
Test Results:
	at com.okada.junit.TestCase.run(TestCase.java:31)
 Failures: 1 Errors: 1
	at com.okada.junit.TestSuite.run(TestSuite.java:23)
There was 1 error:
	at com.okada.junit.TestRunner.run(TestRunner.java:44)
0) test.MathTest.ok
	at com.okada.junit.TestRunner.main(TestRunner.java:37)
There was 1 failure:
	at test.MathTest.main(MathTest.java:31)
0) test.MathTest.add expected:"1" but was:"2"
```

`System.out` 与 `Throwable.printStackTrace()` 的输出内容混在一起了，没有按照顺序来输出

## 原因

问了别人，说是因为 `System.out` 和 `System.err` 的输出顺序问题。于是查看了下 `Throwable.printStackTrace()` 的源码。发现 `Throwable.printStackTrace()` 会调用 `System.err` 进行内容输出

```java
public void printStackTrace() {
    printStackTrace(System.err);
}

public void printStackTrace(PrintStream s) {
    printStackTrace(new WrappedPrintStream(s));
}

private void printStackTrace(PrintStreamOrWriter s) {
    // ...
}
```

再查看网上的解释，`System.out` 是有缓冲区的，JVM 和操作系统会等待缓冲区中的内容达到一定的大小再输出内容。而 `System.err` 的话，是没有缓冲区这种东西，有内容就立刻输出。所以造成了 `System.out` 与 `Throwable.printStackTrace()` 的输出内容混在一起了。这就是原因。

## 如何解决

根据 `Throwable.printStackTrace()` 的源码

```
public void printStackTrace() {
    printStackTrace(System.err);
}

public void printStackTrace(PrintStream s) {
    printStackTrace(new WrappedPrintStream(s));
}

private void printStackTrace(PrintStreamOrWriter s) {
    // ...
}
```

只需要调用 `printStackTrace(PrintStream s)` 传入 `System.err` 即可，也就是调用 `Throwable.printStackTrace(System.err)`。

这样输出内容再也不会顺序错乱了。