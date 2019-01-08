---
title: JS中的函数：形参与实参以及内部原理
date: 2018-09-17 10:48:18
tags:
  - JavaScript
  - 参数传递
categories:
  - JavaScript
---

## 实参个数大于形参个数

对于如下函数

```js
function say(name, msg) {
    console.log(name);
    console.log(msg);
}
```

这句话的调用结果是什么？

```
say('1', '2', '3')
```

<!-- more -->

对于其他语言，比如 Java，要是这么写的话，编译器根本就不会让你编译通过，但是在 JS 里是可以运行的。运行结果是

```js
1
2
undefined
```

## 实参个数小于形参个数

还是同样的代码

```js
function say(name, msg) {
    console.log(name + msg);
}

say('1')
```

打印

```js
1
undefined
```

如果什么参数都不写，直接调用

```js
say();
```

打印

```
undefined
undefined
```

## 结论

> 从上面的例子可以看出，在 JS 中变量定义的时候，如果不给一个变量赋初值那么该变量的类型为 `undefiend`

在进行函数调用的时候，不管实参的数目大于形参还是小于形参被调用的函数都会执行；在JS中函数不介意传递进来多少个参数，也不在乎传进来的参数是什么数据类型。发生函数调用的时候可以给一个实参也可以给多个实参。

## 原理

在 js 中，参数在内部是用一个数组来表示。函数接收到的始终是这个数组，而不关心数组中包含哪些参数，如果这个数组不包含任何参数也无所谓，包含多个参数也没问题。

在函数体内可以通过 `arguments` 对象来访问这个参数数组，从而获取传递给参数的每个参数。

```js
function say(name, msg) {
    for(let i = 0; i < arguments.length; i++) {
        console.log('arg[' + i + '] = ' + arguments[i]);
    }
    console.log('length = ' + arguments.length);
}
```

传入 3 个参数

```js
say('1', '2', '3');
```

打印结果

```
arg[0] = 1
arg[1] = 2
arg[2] = 3
length = 3
```

## 参考

* [JS中的函数(二)：函数参数（你可能不知道的参数传递）](https://www.cnblogs.com/hanhanhan/p/5765920.html)