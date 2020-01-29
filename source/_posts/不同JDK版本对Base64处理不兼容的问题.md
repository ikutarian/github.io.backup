---
title: 不同JDK版本对Base64处理不兼容的问题
date: 2019-12-09 09:20:21
tags:
  - Base64
categories:
  - Java
---


偶然发现使用 JDK8 内置的 `Base64` 解码器解析 Base64 编码的字符串的时候，会抛出 `java.lang.IllegalArgumentException: Illegal base64 character` 异常。这非常奇怪，因为 JDK 是向下兼容的，不应该发生这种情况啊

<!-- more -->