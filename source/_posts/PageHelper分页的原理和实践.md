---
title: PageHelper分页的原理和实践
date: 2018-09-17 10:41:06
tags:
  - MyBatis
  - PageHelper
  - 分页
categories:
  - 
---

## 最终效果

{% asset_img 3617116-6b276f3858b26ef2.png %}

网页上有几个按钮

- 首页
- 上一页
- 第1、2、3...页
- 下一页
- 尾页

<!-- more -->

## 配置

在 `pom.xml` 中加入 PageHelper 的依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.2</version>
</dependency>
```

在mybatis 的配置文件  `mybatis-config.xml` 中引入 PageHelper 插件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 分页插件 -->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!-- 分页参数合理化 -->
            <property name="reasonable" value="true"/>
        </plugin>
    </plugins>

</configuration>
```

## DAO层

只需要写 `SELECT` 语句即可，不用加上 `LIMIT` 限制，比如查询标签列表的 SQL 语句

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.okada.movie.dao.TagDAO">

    <resultMap id="tagResultMap" type="com.okada.movie.model.Tag">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="addtime" property="addTime"/>
    </resultMap>

    <select id="list" resultMap="tagResultMap">
        SELECT
            *
        FROM
            tag
        ORDER BY
            addtime DESC
    </select>

</mapper>
```

## Controller 层

```java
/**
 * 标签列表
 */
@GetMapping("tag/list")
public String tagList(ModelMap map,
                      @RequestParam(value="pageNum", defaultValue="1") int pageNum,
                      @RequestParam(value="pageSize", defaultValue="5") int pageSize) {
    // 只需要调用查询语句之前
    // 在这里填入页码和每页的个数即可实现分页
    PageHelper.startPage(pageNum, pageSize);
    
    List<Tag> tagList = tagService.list();
    PageInfo<Tag> tagPageInfo = new PageInfo<>(tagList);
    map.put("pageInfo", tagPageInfo);

    return "admin/tag_list";
}
```

## 视图层

要实现效果是这样的

![image.png](https://upload-images.jianshu.io/upload_images/3617116-6b276f3858b26ef2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先来撸一撸这几个按钮的逻辑

### 首页

只需要在 url 后面加上页码为 1 的参数即可，比如 `url?pageNum=1`

### 尾页

只需要在 url 后面加上页码为最后一页的参数即可，比如 `url?pageNum=最后一页`

### 上一页

首先判断是否有上一页。有上一页就加上上一页的参数，`url?pageNum=上一页`，没有上一页就把按钮设置为不可点击

### 下一页

和上一页的逻辑同理。判断是否有下一页，有就加上下一页的参数，`url?pageNum=下一页`，没有就把按钮设置为不可点击

### 1、2、3...

获取总页数，然后从第一页开始循环到最后一页。在循环的过程中，如果页码是当前页的话就高亮显示并且不可点击，否则按钮为正常状态

### 如果只有一页数据

如果只有一页数据，那么分页也就没有存在的必要。分页的视图就没有必要显示了

### JSP代码

PageHelper 已经封装了是否有上一页、是否有下一页、总页数、当前页等分页的信息，所以可以直接拿来用。

如果用 JSP 来实现的话，如下

```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<c:if test="${pageInfo.getPages() > 1}">
    <ul class="pagination pagination-sm no-margin pull-right">
        <li><a href="?pageNum=1">首页</a></li>

        <c:choose>
            <c:when test="${pageInfo.isHasPreviousPage() == true}">
                <li><a href="?pageNum=${pageInfo.getPrePage()}">上一页</a></li>
            </c:when>
            <c:otherwise>
                <li class="disabled"><a href="#">上一页</a></li>
            </c:otherwise>
        </c:choose>

        <c:forEach var="page" begin="1" end="${pageInfo.getPages()}" step="1">
            <c:choose>
                <c:when test="${pageInfo.getPageNum() == page}">
                    <li class="active"><a href="#">${page}</a></li>
                </c:when>
                <c:otherwise>
                    <li><a href="?pageNum=${page}">${page}</a></li>
                </c:otherwise>
            </c:choose>
        </c:forEach>

        <c:choose>
            <c:when test="${pageInfo.isHasNextPage() == true}">
                <li><a href="?pageNum=${pageInfo.getNextPage()}">下一页</a></li>
            </c:when>
            <c:otherwise>
                <li class="disabled"><a href="#">下一页</a></li>
            </c:otherwise>
        </c:choose>

        <li><a href="?pageNum=${pageInfo.getPages()}">尾页</a></li>
    </ul>
</c:if>
```

## 总结

* 理清楚分页的逻辑
* PageHelper 的使用