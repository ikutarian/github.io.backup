---
title: '@WebServlet失效的问题'
date: 2018-09-27 11:24:39
tags:
  - Servlet
  - web.xml
  - 注解
categories:
  - JavaWeb
---

我创建了一个 Servlet，利用 `@WebServlet` 注解指定 url，但是在浏览器中访问不到，显示的是 404 页面

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String currentTime = dateFormat.format(new Date());
        req.setAttribute("currentTime", currentTime);
        req.getRequestDispatcher("/WEB-INF/jsp/hello.jsp").forward(req, resp);
    }
}
```

<!-- more -->

检查了一下我的 `web.xml` 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1"
         metadata-complete="true">

</web-app>
```

目前是空白的，什么都没有。但是访问 `http://localhost:8080/hello` 还是访问不到。于是我就修改 `web.xml`，由 `web.xml` 来指定路由

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1"
         metadata-complete="true">

    <servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.smart4j.chapter1.HelloServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>

</web-app>
```

现在可以访问了。

## 原因

查找资料，问题出在 `web.xml` 文件中。xml 文件定义中的 `metadata-complete="true"`，造成了注解失效。

> 该属性指定当前的部署描述文件是否是完全的。如果设置为 `true`，则容器在部署时将只依赖部署描述文件，忽略所有的注解（同时也会跳过 `web-fragment.xml` 的扫描），如果不配置该属性，或者将其设置为 `false`，则表示启用注解支持（和 `web-fragment.xml` 的扫描）