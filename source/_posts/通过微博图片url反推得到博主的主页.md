---
title: 通过微博图片url反推得到博主的主页
date: 2018-09-17 10:53:19
tags:
  - 微博
  - 62进制
  - 进制转换
categories:
  - Hack
---

## url的生成规则

在v2ex发现了一个帖子

{% asset_img 3617116-39ef9800d0915e29.png %}

<!-- more -->

## 代码

于是，我写了一个根据url反推博主主页的小工具 `WeiboHack`

```java
public class WeiBoHack {

    public static String getIndexPage(String picUrl) {
        String fileName = picUrl.substring(picUrl.lastIndexOf("/") + 1, picUrl.lastIndexOf("."));
        if (fileName.startsWith("00")) {
            String front8Letter = fileName.substring(0, 8);
            long uuid = Base62Utils.decodeBase62(front8Letter);
            return appendUUID(uuid);
        } else {
            String front8Letter = fileName.substring(0, 8);
            long uuid = Long.valueOf(front8Letter, 16);
            return appendUUID(uuid);
        }
    }

    private static String appendUUID(long uuid) {
        return "https://weibo.com/u/" + uuid;
    }
}
```

`Base62Utils` 的工具类代码如下

```java
public class Base62Utils {

    private static final String base62Char = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    public static long decode(String str) {
        char[] chars = new StringBuilder(str).reverse().toString().toCharArray();
        long count = 1;
        long result = 0;
        for (char aChar : chars) {
            result += base62Char.indexOf(aChar) * count;
            count *= 62;
        }
        return result;
    }
}
```