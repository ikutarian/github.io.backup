---
title: SimpleDateFormat的线程安全问题与ThreadLocal
date: 2018-09-17 14:04:50
tags:
  - SimpleDateFormat
  - ThreadLocal
  - 线程安全
  - 多线程
categories:
  - 多线程
---

查看 `SimpleDateFormat` 的文档注释

> SimpleDateFormat is not thread-safe. Users should create a separate instance for each thread.

说 `SimpleDateFormat` 不是线程安全的，需要为每一个线程创建一个单独的实例来用。为什么是线程不安全的？写个例子试试就知道。

<!-- more -->

下面这个例子，创建了一个 `SimpleDateFormat` 实例，然后格式化同一个时间，如果格式化出来的结果不一致的或者抛出了异常，就证明 `SimpleDateFormat` 确实是线程不安全的。

```java
public class SimpleDateFormatTest {

	public static void main(String[] args) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		String dateTime = "2016-12-30 15:35:34";
		for (int i = 0; i < 5; i++) {
			new Thread(new Runnable() {
				@Override
				public void run() {
					for (int i = 0; i < 5; i++) {
						try {
							System.out.println(
									Thread.currentThread().getName() + "\t" + dateFormat.parse(dateTime));
						} catch (ParseException e) {
							e.printStackTrace();
						}
					}
				}
			}).start();
		}
	}
}
```

下面是打印结果。可以发现，不光是抛出了异常，而且打印出来的结果存在不一致的现象。看来，`SimpleDateFormat` 果然是线程不安全的。

```
Exception in thread "Thread-2" Exception in thread "Thread-0" java.lang.NumberFormatException: For input string: ""
	at java.lang.NumberFormatException.forInputString(Unknown Source)
	at java.lang.Long.parseLong(Unknown Source)
	at java.lang.Long.parseLong(Unknown Source)
	at java.text.DigitList.getLong(Unknown Source)
	at java.text.DecimalFormat.parse(Unknown Source)
	at java.text.SimpleDateFormat.subParse(Unknown Source)
	at java.text.SimpleDateFormat.parse(Unknown Source)
	at java.text.DateFormat.parse(Unknown Source)
	at com.owen.encrypt.SimpleDateFormatTest$1.run(SimpleDateFormatTest.java:18)
	at java.lang.Thread.run(Unknown Source)
Exception in thread "Thread-1" java.lang.NumberFormatException: multiple points
	at sun.misc.FloatingDecimal.readJavaFormatString(Unknown Source)
	at sun.misc.FloatingDecimal.parseDouble(Unknown Source)
	at java.lang.Double.parseDouble(Unknown Source)
	at java.text.DigitList.getDouble(Unknown Source)
	at java.text.DecimalFormat.parse(Unknown Source)
	at java.text.SimpleDateFormat.subParse(Unknown Source)
	at java.text.SimpleDateFormat.parse(Unknown Source)
	at java.text.DateFormat.parse(Unknown Source)
	at com.owen.encrypt.SimpleDateFormatTest$1.run(SimpleDateFormatTest.java:18)
	at java.lang.Thread.run(Unknown Source)
java.lang.NumberFormatException: multiple points
	at sun.misc.FloatingDecimal.readJavaFormatString(Unknown Source)
	at sun.misc.FloatingDecimal.parseDouble(Unknown Source)
	at java.lang.Double.parseDouble(Unknown Source)
	at java.text.DigitList.getDouble(Unknown Source)
	at java.text.DecimalFormat.parse(Unknown Source)
	at java.text.SimpleDateFormat.subParse(Unknown Source)
	at java.text.SimpleDateFormat.parse(Unknown Source)
	at java.text.DateFormat.parse(Unknown Source)
	at com.owen.encrypt.SimpleDateFormatTest$1.run(SimpleDateFormatTest.java:18)
	at java.lang.Thread.run(Unknown Source)
Thread-3	Tue Dec 30 15:35:34 CST 4
Thread-3	Fri Dec 30 15:35:34 CST 2016
Thread-3	Fri Dec 30 15:35:34 CST 2016
Thread-3	Fri Dec 30 15:35:34 CST 2016
Thread-3	Fri Dec 30 15:35:34 CST 2016
Thread-4	Fri Dec 30 15:35:34 CST 42016
Thread-4	Fri Dec 30 15:35:34 CST 2016
Thread-4	Fri Dec 30 15:35:34 CST 2016
Thread-4	Fri Dec 30 15:35:34 CST 2016
Thread-4	Fri Dec 30 15:35:34 CST 2016
```

## 什么原因造成 SimpleDateFormat 是线程不安全的？

看一下 `SimpleDateFormat` 的源码。在 `SimpleDateFormat` 的父类 `DateFormat` 中可以看到一个成员变量

```java
/**
 * The {@link Calendar} instance used for calculating the date-time fields
 * and the instant of time. This field is used for both formatting and
 * parsing.
 *
 * <p>Subclasses should initialize this field to a {@link Calendar}
 * appropriate for the {@link Locale} associated with this
 * <code>DateFormat</code>.
 * @serial
 */
protected Calendar calendar;
```
注释说了，calendar 是用在 format 和 parse 时用的。另外，因为 calendar 作为一个成员变量，在多线程场景下，会发生资源共享造成前后不一致的问题。这就是 `SimpleDateFormat` 是线程不安全的原因。

## 如何避免

### 方法的局部变量

有两种方法，写一个单独的方法，然后将 `SimpleDateFormat` 作为方法的成员变量，每个线程需要格式化时间的时候，就去调用这个方法，`SimpleDateFormat` 作为方法的成员变量，自然就不存在资源共享的问题了。

```java
public static String getCurrentTime(String format) {
	Date date = new Date();
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat(format, Locale.getDefault());
	return simpleDateFormat.format(date);
}
```

但是每次调用这个方法就去 new 一个 `SimpleDateFormat` 对性能来说也是一个开销。

### 使用 ThreadLocal

第二种方法是使用 `ThreadLocal` 来存放 `SimpleDateFormat`。`ThreadLocal` 的特性决定了每个线程操作 `ThreadLocal` 中的值，不会影响到别的线程

```java
/**
 * 时间工具类
 */
public class TimeUtil {

    public static final String YEAR_MONTH_DAY_SECOND = "yyyy-MM-dd HH:mm:ss";
    public static final String YEAR_MONTH_DAY_SECOND2 = "yyyy/MM/dd HH:mm:ss";
    public static final String YEAR_MONTH_DAY_SECOND3 = "yyyy年MM月dd日 HH时mm分ss秒";
    public static final String YEAR_MONTH_DAY = "yyyy-MM-dd";
    public static final String YEAR_MONTH_DAY2 = "yyyy年MM月dd日";

    /**
     * 采用 ThreadLocal 避免 SimpleDateFormat 非线程安全的问题
     * <p>
     * Key —— 时间格式
     * Value —— 解析特定时间格式的 SimpleDateFormat
     */
    private static ThreadLocal<Map<String, SimpleDateFormat>> sThreadLocal = new ThreadLocal<>();

    /**
     * 获取解析特定时间格式的 SimpleDateFormat
     *
     * @param pattern 时间格式
     */
    private static SimpleDateFormat getDateFormat(String pattern) {
        Map<String, SimpleDateFormat> strDateFormatMap = sThreadLocal.get();

        if (strDateFormatMap == null) {
            strDateFormatMap = new HashMap<>();
        }

        SimpleDateFormat simpleDateFormat = strDateFormatMap.get(pattern);
        if (simpleDateFormat == null) {
            simpleDateFormat = new SimpleDateFormat(pattern, Locale.getDefault());
            strDateFormatMap.put(pattern, simpleDateFormat);
			sThreadLocal.put(strDateFormatMap);
        }

        return simpleDateFormat;
    }

    /**
     * 时间格式化
     *
     * @param date：要格式化的时间
     * @param pattern：要格式化的类型
     */
    public static String formatDate(Date date, String pattern) {
        if (date == null || pattern == null) {
            return null;
        }

        return getDateFormat(pattern).format(date);
    }
}
```
## 参考来源

http://blog.jrwang.me/2016/java-simpledateformat-multithread-threadlocal/