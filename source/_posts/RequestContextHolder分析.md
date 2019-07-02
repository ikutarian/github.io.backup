---
title: RequestContextHolder分析
date: 2019-07-02 10:42:20
tags:
  - RequestContextHolder
  - HttpServletRequest
  - SpringMVC
  - ThreadLocal
categories:
  - Spring
---

在 Service 层或者某一个工具类中获取到 `HttpServletRequest` 对象的场景还是比较常见的。一种方法是将 `HttpServletRequest` 对象作为方法的入参从 Controller 层一直往下传递。不过这样比较费劲，不够优雅。还有一种就是利用 `RequestContextHolder`，在需要 `HttpServletRequest` 的时候，用如下方式获取

```java
package io.renren.common.utils;

import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import javax.servlet.http.HttpServletRequest;

public class HttpContextUtils {

    /**
     * 获取HttpServletReqest对象
     */
	public static HttpServletRequest getHttpServletRequest() {
        return ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
	}
}
```

<!-- more -->

要理解为什么可以这么使用，需要明白两个问题：

1. `RequestContextHolder` 为什么能获取到当前的 `HttpServletRequest` ？
2. `HttpServletRequest` 是在什么时候设置到 `RequestContextHolder` 的？

对于第 1 个问题，熟悉 `ThreadLocal` 的人应该很容易看出来这个是利用 `ThreadLocal` 获取到的。第 2 哥问题应该是属于 SpringMVC 的问题

# 源码分析

首先看一下 `RequestContextHolder` 的源码

```java
public abstract class RequestContextHolder  {

	private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
			new NamedThreadLocal<>("Request attributes");

	private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
			new NamedInheritableThreadLocal<>("Request context");


	/**
	 * Reset the RequestAttributes for the current thread.
	 */
	public static void resetRequestAttributes() {
		requestAttributesHolder.remove();
		inheritableRequestAttributesHolder.remove();
	}

	/**
	 * Bind the given RequestAttributes to the current thread,
	 * <i>not</i> exposing it as inheritable for child threads.
	 * @param attributes the RequestAttributes to expose
	 * @see #setRequestAttributes(RequestAttributes, boolean)
	 */
	public static void setRequestAttributes(@Nullable RequestAttributes attributes) {
		setRequestAttributes(attributes, false);
	}

	/**
	 * Bind the given RequestAttributes to the current thread.
	 * @param attributes the RequestAttributes to expose,
	 * or {@code null} to reset the thread-bound context
	 * @param inheritable whether to expose the RequestAttributes as inheritable
	 * for child threads (using an {@link InheritableThreadLocal})
	 */
	public static void setRequestAttributes(@Nullable RequestAttributes attributes, boolean inheritable) {
		if (attributes == null) {
			resetRequestAttributes();
		}
		else {
			if (inheritable) {
				inheritableRequestAttributesHolder.set(attributes);
				requestAttributesHolder.remove();
			}
			else {
				requestAttributesHolder.set(attributes);
				inheritableRequestAttributesHolder.remove();
			}
		}
	}

	/**
	 * Return the RequestAttributes currently bound to the thread.
	 * @return the RequestAttributes currently bound to the thread,
	 * or {@code null} if none bound
	 */
	@Nullable
	public static RequestAttributes getRequestAttributes() {
		RequestAttributes attributes = requestAttributesHolder.get();
		if (attributes == null) {
			attributes = inheritableRequestAttributesHolder.get();
		}
		return attributes;
	}

}
```

`setRequestAttributes()` 方法的作用就是：将 `RequestAttributes` 对象放入到 `ThreadLocal` 中。而 `HttpServletRequest`和 `HttpServletResponse` 等则封装在 `RequestAttributes` 对象中，在此处就不对 `RequestAttributes` 这个类展开。反正我们需要知道的就是要获取 `RequestAttributes` 对象，然后再从 `RequestAttributes` 对象中获取到我们所需要的 `HttpServletRequest` 即可

那么在 SpringMVC 中是怎么实现的呢，看看 `DispatcherServlet` 的的父类 `FrameworkServlet` 的源码。

```java
/**
 * Process this request, publishing an event regardless of the outcome.
 * <p>The actual event handling is performed by the abstract
 * {@link #doService} template method.
 */
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    /*
     * 创建ServletRequestAttributes对象
     */
    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    /*
     * 将RequestAttributes设置到RequestContextHolder中
     */
    initContextHolders(request, localeContext, requestAttributes);

    try {
        /*
         * 传入HttpServletRequest与HttpServletResponse，实现具体的业务逻辑
         */
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        /*
         * 重置RequestContextHolder
         */
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

简单看下源码，我们可以知道 `HttpServletRequest` 是在执行 `doService` 方法之前，也就是具体的业务逻辑前进行设置的，然后在执行完业务逻辑或者抛出异常时重置 `RequestContextHolder` 移除当前的 `HttpServletRequest`

写本文的目的主要是记录下这个 `RequestContextHolder` 的使用，以及希望以后能在业务代码中巧用该工具让自己的代码更加简洁优雅