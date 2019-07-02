---
title: renren-fast项目的分析
date: 2019-06-13 11:42:03
tags:
  - renren-fast
categories:
  - 项目分析
---

[renren-fast](https://gitee.com/renrenio/renren-fast) 是一个很好的项目，值得学习

<!-- more -->

# 登陆

前后端分离时，验证码与账户验证如何处理？

## 验证码

### 思路

没有前后端分离的时候，是这么做的：

1. 后端生成验证码时，把验证码的值存放到 Session 中
2. 登陆时后端把前端传回的表单中的验证码与 Session 中存放的验证码进行比对，如果一致就放行，不一致就拦截

现在前后端分离了，后端没办法在 Session 中存储验证码了，这时候就要换一个思路：

1. 前端生成一个随机字符串，这个字符串要求唯一
2. 前端带上这个随机字符串请求后端的获取验证码接口
3. 后端生成验证码，并将随机字符串与验证码存储起来，可以存到数据库，也可以存到 Redis 中
4. 前端登陆时，带上验证码与随机字符串。后端根据随机字符串取出验证码，再与前端传过来的验证码进行比对

### 表结构

使用数据库存储随机字符串与验证码，表结构如下

```sql
-- 系统验证码
CREATE TABLE `sys_captcha` (
  `uuid` char(36) NOT NULL COMMENT 'uuid',
  `code` varchar(6) NOT NULL COMMENT '验证码',
  `expire_time` datetime DEFAULT NULL COMMENT '过期时间',
  PRIMARY KEY (`uuid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='系统验证码';
```

如果使用 Redis 存储验证码，可以以随机字符串为 key，验证码值为 value，然后再设置一个过期时间。假设随机字符是 `2d88a4af-01a4-41f3-adb3-5ba31a7edc7c`，验证码是 `asd123`，5 分钟内有效，Redis 的命令如下

```
SET 2d88a4af-01a4-41f3-adb3-5ba31a7edc7c asd123 EX 300
```

