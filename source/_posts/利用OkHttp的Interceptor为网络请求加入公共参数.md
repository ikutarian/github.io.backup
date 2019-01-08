---
title: 利用OkHttp的Interceptor为网络请求加入公共参数
date: 2018-09-15 15:07:36
tags:
  - OkHttp
  - 公共参数
  - Interceptor
categories:
  - Android
---

## 背景

在开发中经常遇到这样的需求：每一次网络请求带上几个固定的参数，例如：

平台：os=android
API版本: v=1.0
...

这些参数就是为公共参数。公共参数一般是加在 Header，URL Query 或者 POST body 中。

<!-- more -->

## 实现

我用的网络请求库是 OkHttp，它提供了一个很好用的拦截器（Interceptors），只需要实现这个拦截器的方法，然后加入到 http client 中即可。

## 使用

公共参数拦截器 CommonParamsInterceptor 向外暴露了3个方法，只需要将公共参数作为键值对传入即可

```
private RxRequestHelper(Interceptor commonParamsInterceptor) {
	OkHttpClient.Builder clientBuilder = new OkHttpClient.Builder();

	// 公共参数
	if (commonParamsInterceptor != null) {
		clientBuilder.addInterceptor(new CommonParamsInterceptor() {
            @Override
            public Map<String, String> getHeaderMap() {  // Header
                Map<String, String> headersMap =  new HashMap<>();
                headersMap.put("v", "1.0");
                return headersMap;
            }

            @Override
            public Map<String, String> getQueryParamMap() { // URL Query
                Map<String, String> queryParamMap =  new HashMap<>();
                queryParamMap.put("v", "1.0");
                return queryParamMap;
            }

            @Override
            public Map<String, String> getFormBodyParamMap() { // POST body
                Map<String, String> formBodyParamMap =  new HashMap<>();
                formBodyParamMap.put("v", "1.0");
                return formBodyParamMap;
            }
        });
	}

	sRetrofit = new Retrofit.Builder()
			.baseUrl(BuildConfig.BASE_SERVER_URL)
			.addConverterFactory(GsonConverterFactory.create())
			.addCallAdapterFactory(RxJavaCallAdapterFactory.create())
			.client(clientBuilder.build())
			.build();
}
```
## 源码

https://github.com/ikutarian/CommonParamsInterceptor

如果对你有用，请帮我点一个 star 吧

## 参考

在代码实现的过程中，参考了[BasicParamsInterceptor - 为 OkHttp 请求添加公共参数](http://www.jianshu.com/p/0c1bc054b293)，然后在其基础上实现了它没有实现的对 **POST form-data** 参数的封装
