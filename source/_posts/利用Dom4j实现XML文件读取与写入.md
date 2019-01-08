---
title: 利用Dom4j实现XML文件读取与写入
date: 2019-01-06 21:31:27
tags:  
  - XML
  - Dom4j
categories:
  - Java
---

# Maven依赖

```xml
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
```

<!-- more -->

# 读取

## 文件内容

一个名为 `books.xml` 的 XML 文件放在 `resources` 文件夹中，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
    <book id="1">
        <name>冰与火之歌</name>
        <author>乔治马丁</author>
        <year>2014</year>
        <price>89</price>
    </book>
    <book id="2">
        <name>安徒生童话</name>
        <author>2004</author>
        <price>77</price>
        <language>English</language>
    </book>
</bookstore>
```

## 代码

```java
import org.dom4j.Attribute;
import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.util.Iterator;
import java.util.List;

public class HelloWorld {

    public static void main(String[] args) throws Exception {
        // 读取XML文件
        SAXReader reader = new SAXReader();
        Document document = reader.read(HelloWorld.class.getClassLoader().getResourceAsStream("books.xml"));
        // 获取根节点
        Element bookstore = document.getRootElement();
        // 获取子节点的迭代器
        Iterator iterator = bookstore.elementIterator();
        // 遍历迭代器
        while (iterator.hasNext()) {
            System.out.println("---------------------------------");

            Element book = (Element) iterator.next();
            // 获取属性名与属性值
            List bookAttrs = book.attributes();
            for (Object attr : bookAttrs) {
                Attribute bookAttr = (Attribute) attr;
                System.out.println("属性名: " + bookAttr.getName() +
                        ", 属性值: " + bookAttr.getValue());
            }

            // 获取book节点的子节点
            Iterator itt = book.elementIterator();
            while (itt.hasNext()) {
                Element bookChild = (Element) itt.next();
                System.out.println("节点名: " + bookChild.getName()
                        + ", 节点值: " + bookChild.getStringValue());
            }
        }
    }
}
```

输出结果：

```
---------------------------------
属性名: id, 属性值: 1
节点名: name, 节点值: 冰与火之歌
节点名: author, 节点值: 乔治马丁
节点名: year, 节点值: 2014
节点名: price, 节点值: 89
---------------------------------
属性名: id, 属性值: 2
节点名: name, 节点值: 安徒生童话
节点名: author, 节点值: 2004
节点名: price, 节点值: 77
节点名: language, 节点值: English
```

# 写入

## 生成结果

比如要生成这样一份 XML 文件，内容如下：

```xml
<?xml version="1.0" encoding="GBK"?>

<rss version="2.0">
  <channel>
    <title><![CDATA[上海移动互联网产业促进中心正式揭牌 ]]></title>
  </channel>
</rss>
```

## 代码

```java
import org.dom4j.Document;
import org.dom4j.DocumentHelper;
import org.dom4j.Element;
import org.dom4j.io.OutputFormat;
import org.dom4j.io.XMLWriter;

import java.io.FileOutputStream;

public class HelloWorld {

    public static void main(String[] args) throws Exception {
        // 创建Document对象，Document对象表示整个XML文档
        Document document = DocumentHelper.createDocument();
        // 创建根节点
        Element rss = document.addElement("rss");
        // 添加属性
        rss.addAttribute("version", "2.0");
        // 生成子节点及子节点内容
        rss.addElement("channel").addElement("title")
                .addCDATA("上海移动互联网产业促进中心正式揭牌");  // 对于CDATA内容可以调用addCDATA方法
        // 保存到文件
        OutputFormat outputFormat = OutputFormat.createPrettyPrint();
        outputFormat.setEncoding("GBK");
        XMLWriter writer = new XMLWriter(new FileOutputStream("F:/rss.xml"), outputFormat);
        writer.write(document);
        writer.close();
    }
}
```

