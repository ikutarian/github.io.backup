---
title: 使用OkHttp调用Webservice服务
date: 2019-06-04 21:36:14
tags:
  - OkHttp
  - Webservice
  - SOAP
categories:
  - Java
---

工作中遇到需要请求 Webservice 的场景。如果用一些 WebService 框架去请求的话需要引入很多的库。这样太麻烦了，学习也要时间。反正请求 Webservice 其实也是发起 HTTP 请求，就用 HTTP 库来实现好了。于是就用 OkHttp 来发起请求调用 Webservice 服务

<!-- more -->

# 报文格式是什么样的？

报文格式可以使用 [SoapUI](https://www.soapui.org/) 来查看。SoapUI 和常用的 PostMan 一样都是接口测试工具。不过 SoapUI 在调用 WebService 时更专业一点

现在有一个天气预报的 WebService WSDL：http://www.webxml.com.cn/WebServices/WeatherWS.asmx?wsdl。打开 SoapUI，选择 “File - New SOAP Project”

{% asset_img Snipaste_2019-06-11_10-30-19.png %}

填写项目名称和 WSDL，然后点击 “OK”

{% asset_img Snipaste_2019-06-11_10-31-15.png %}

就会在侧边栏看到项目了

{% asset_img Snipaste_2019-06-11_10-32-20.png %}

随便点击一个接口，比如 `getSupportCity`，就可以看到请求报文格式了

{% asset_img Snipaste_2019-06-11_10-36-10.png %}

现在按照文档，填充请求报文，点击左上角的绿色三角发起请求

{% asset_img Snipaste_2019-06-11_10-37-19.png %}

现在就可以看到返回的报文了

{% asset_img Snipaste_2019-06-11_10-37-37.png %}

那么报文格式呢？点击这个 “RAW” 按钮就可看到了

{% asset_img Snipaste_2019-06-11_10-41-08.png %}

**需要关注的是 Header 部分**，这两个最重要

```
Content-Type: text/xml;charset=UTF-8
SOAPAction: "http://WebXml.com.cn/getSupportCity"
```

现在知道报文格式了，那么就可以利用 PostMan 来组装报文请求 WebService 试试

# 使用 PostMan 组装报文请求 WebService

填写 URL，使用 Post 请求，包体选择 Raw，并选择 `XML(text/xml)`，填写报文

{% asset_img Snipaste_2019-06-11_10-45-30.png %}

填写 Header

{% asset_img Snipaste_2019-06-11_10-46-29.png %}

现在发起请求试试，可以看到有响应报文返回

{% asset_img Snipaste_2019-06-11_10-47-46.png %}

# 使用 OkHttp 请求 WebService

引入依赖

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.14.2</version>
</dependency>
```

代码如下

```java
String url = "http://www.webxml.com.cn/WebServices/WeatherWebService.asmx";
String requestBody = "<soapenv:Envelope xmlns:soapenv=\"http://schemas.xmlsoap.org/soap/envelope/\" xmlns:web=\"http://WebXml.com.cn/\">\n" +
                "   <soapenv:Header/>\n" +
                "   <soapenv:Body>\n" +
                "      <web:getSupportCity>\n" +
                "         <!--Optional:-->\n" +
                "         <web:byProvinceName>福建</web:byProvinceName>\n" +
                "      </web:getSupportCity>\n" +
                "   </soapenv:Body>\n" +
                "</soapenv:Envelope>";;

OkHttpClient client = new OkHttpClient();
MediaType mediaType = MediaType.parse("text/xml");
RequestBody body = RequestBody.create(mediaType, requestBody);
Request request = new Request.Builder()
        .url(url)
        .post(body)
        .addHeader("SOAPAction", "http://WebXml.com.cn/getSupportCity")
        .addHeader("Content-Type", "text/xml; charset=utf-8")
        .build();
Response response = client.newCall(request).execute();
if (response.isSuccessful()) {
    String xmlResult = response.body().string();
    log.info("SOAP请求成功，返回内容为: {}", xmlResult);
} else {
    log.error("SOAP请求失败，HTTP状态码为：{}", response.code());
}
```

格式就是这样，剩下的就是根据项目进行封装了

# 利用 PostMan 生成代码

如果连代码都懒得写的话，可以用 PostMan 生成代码。点击 “Code”

{% asset_img Snipaste_2019-06-11_11-22-09.png %}

选择 “Java - OK HTTP” 就可以生成 OkHTTP 的 Java 代码了

{% asset_img Snipaste_2019-06-11_11-22-34.png %}

只不过这部分的代码可以酌情删除

{% asset_img Snipaste_2019-06-11_11-24-01.png %}

# 总结

以上就是如何利用 SoapUI 分析报文，利用 PostMan 组装报文，使用 OkHttp 请求 WebService 的全部内容了