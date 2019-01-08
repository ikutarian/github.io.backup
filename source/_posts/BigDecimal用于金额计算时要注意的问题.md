---
title: BigDecimal用于金额计算时要注意的问题
date: 2018-11-07 09:15:16
tags:
  - BigDecimal
  - 金额计算
categories:
  - Java
---

## 为什么要用 BigDecimal

由于计算机底层是二进制计算，所以浮点型运算会出现精度问题，比如

```java
public class HelloWorld {

    public static void main(String[] args) {
        System.out.println(0.2 + 0.1);
        System.out.println(0.3 - 0.1);
        System.out.println(0.2 * 0.1);
    }
}
```

输出的结果是

```
0.30000000000000004
0.19999999999999998
0.020000000000000004
```

这样的计算结果在金额计算中肯定是无法接受的，所以就有了 BigDecimal

<!-- more -->

## 改成使用 BigDecimal 进行浮点型运算

把上面的例子改成使用 BigDecimal 的话

```java
import java.math.BigDecimal;

public class HelloWorld {

    public static void main(String[] args) {
        BigDecimal n1 = new BigDecimal("0.1");
        BigDecimal n2 = new BigDecimal("0.2");
        BigDecimal n3 = new BigDecimal("0.3");

        System.out.println(n2.add(n1));
        System.out.println(n3.min(n1));
        System.out.println(n2.multiply(n1));
    }
}
```

输出

```
0.3
0.1
0.02
```

现在计算结果就是正确的了

## BigDecimal 也有精度问题

把代码改成这样

```java
import java.math.BigDecimal;

public class HelloWorld {

    public static void main(String[] args) {
        BigDecimal n1 = new BigDecimal(0.1);
        BigDecimal n2 = new BigDecimal(0.2);
        BigDecimal n3 = new BigDecimal(0.3);

        System.out.println(n2.add(n1));
        System.out.println(n3.min(n1));
        System.out.println(n2.multiply(n1));
    }
}
```

输出的结果出乎意料

```
0.3000000000000000166533453693773481063544750213623046875
0.1000000000000000055511151231257827021181583404541015625
0.0200000000000000022204460492503131424770215565731879227912941627176741932192527428924222476780414581298828125
```

现在 BigDecimal 也有精度问题了。为什么呢？

## 选择正确的构造方法

**重要**

应该选择使用 String 类型参数的构造方法

```
public BigDecimal(String val)
```

所以在编码时，可以这么使用

```
BigDecimal n1 = new BigDecimal("0.1");
BigDecimal n2 = new BigDecimal(Double.toString(0.2));
```

要么直接传入字符串，要么使用 `Double.toString()` 把数字转换成字符串

## BigDecimal 的除法运算

BigDecimal 除法可能出现不能整除的情况。比如 1 / 0.3 会抛出异常

```java
import java.math.BigDecimal;

public class HelloWorld {

    public static void main(String[] args) {
        BigDecimal n1 = new BigDecimal("1");
        BigDecimal n3 = new BigDecimal("0.3");

        System.out.println(n1.divide(n3));
    }
}
```

输出

```
Exception in thread "main" java.lang.ArithmeticException: Non-terminating decimal expansion; no exact representable decimal result.
	at java.math.BigDecimal.divide(BigDecimal.java:1690)
	at com.ikutarian.logback.HelloWorld.main(HelloWorld.java:11)
```

这时候，就需要调用三个参数的 `divide` 方法

```java
public BigDecimal divide(BigDecimal divisor, int scale, int roundingMode)
```

`divisor` 表示除数， `scale` 表示小数点后保留位数，`roundingMode` 表示舍入模式，只有在作除法运算或四舍五入时才用到舍入模式，有下面这几种

- ROUND_UP
- ROUND_DOWN
- ROUND_CEILING
- ROUND_FLOOR
- ROUND_HALF_UP
- ROUND_HALF_DOWN
- ROUND_HALF_EVEN
- ROUND_UNNECESSARY

金额计算一般是用“四舍五入”，所以采用 `ROUND_HALF_UP` 即可。

所以上面的代码应该改成


```java
import java.math.BigDecimal;

public class HelloWorld {

    public static void main(String[] args) {
        BigDecimal n1 = new BigDecimal("1");
        BigDecimal n3 = new BigDecimal("0.3");

        System.out.println(n1.divide(n3, 2, BigDecimal.ROUND_HALF_UP));
    }
}
```

这时候就能输出保留 2 位小数、并且四舍五入的结果了。而且没有抛出异常

```
3.33
```

## 总结

1. 选择使用 String 类型参数的构造方法
2. 除法运算时，要调用三个参数的 `divide` 方法