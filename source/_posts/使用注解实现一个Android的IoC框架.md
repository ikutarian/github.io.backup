---
title: 使用注解实现一个Android的IoC框架
date: 2018-09-17 14:07:40
tags:
  - IoC
  - 注解
  - 反射
categories:
  - Android
---

利用所学的注解和反射的知识，实现一个 IoC 框架，简化代码。

<!-- more -->

最终的效果如下

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends AppCompatActivity {

    @ViewInject(R.id.tv)
    TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Views.inject(this);
    }

    @OnClick(R.id.btn)
    void showToast(View v) {
        mTextView.setText(String.valueOf(new Random().nextInt()));
        Log.d("MainActivity", "showToast: " + (v.getId() == R.id.btn));
    }
}
```

## 什么是 IoC

IoC，就是控制反转（Inversion of Control），缩写为 IoC。

简单来说，就是不需要手动 new 对象，只需要在配置文件或者注解指定一下类名，让框架去解析配置文件或者注解，然后利用反射实例化我们配置的类。

比如，在 Android 应用开发中，我们需要手动调用 `setContentView(int layoutResID)`、`findViewById(int id)` 和 `setOnClickListener(OnClickListener l)` 方法，如果利用注解，只需要指定一下布局文件或者控件的资源 Id，让框架办公我们去找到空间和调用方法。比如下面的代码就是一个例子

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends AppCompatActivity {

    @ViewInject(R.id.tv)
    TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Views.inject(this);
    }

    @OnClick(R.id.btn)
    void showToast(View v) {
        mTextView.setText(String.valueOf(new Random().nextInt()));
        Log.d("MainActivity", "showToast: " + (v.getId() == R.id.btn));
    }
}
```

## 实现

需要三个注解：

### 注解布局文件

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ContentView {
    int NONE = -1;
    int value() default NONE;
}
```

### 注解视图

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ViewInject {
    int NONE = -1;
    int value() default NONE;
}
```

### 注解点击事件

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OnClick {
    int NONE = -1;
    int value();
}
```

有了注解，接着就需要一个注解解析类，去解析被注解的类，得到信息，然后调用相关的方法

```java
public class Views {

    public static void inject(final Activity activity) {
        injectContentView(activity);
        injectViews(activity);
        injectOnClickListener(activity);
    }
}
```

这个注解解析类要做的事情有三件：

1. 解析布局文件的注解
2. 解析视图的注解
3. 解析点击事件的注解

### 解析布局文件的注解

```java
private static void injectContentView(Activity activity) {
	Class<? extends Activity> activityClass = activity.getClass();
	ContentView contentViewAnnotation = activityClass.getAnnotation(ContentView.class);
	if (contentViewAnnotation == null) {
		return;
	}
	int layoutResId = contentViewAnnotation.value();
	if (layoutResId == ContentView.NONE) {
		return;
	}
	activity.setContentView(layoutResId);
}
```

### 解析视图的注解

```java
private static void injectViews(Activity activity) {
	Class<? extends Activity> activityClass = activity.getClass();
	Field[] fields = activityClass.getDeclaredFields();
	for (Field field : fields) {
		ViewInject viewInjectAnnotation = field.getAnnotation(ViewInject.class);
		if (viewInjectAnnotation == null) {
			continue;
		}
		int viewResId = viewInjectAnnotation.value();
		if (viewResId != ViewInject.NONE) {
			try {
				View view = activity.findViewById(viewResId);
				field.setAccessible(true);
				field.set(activity, view);
			} catch (IllegalAccessException e) {
				e.printStackTrace();
			}
		}
	}
}
```

### 解析点击事件的注解

```java
private static void injectOnClickListener(final Activity activity) {
	Class<? extends Activity> activityClass = activity.getClass();
	Method[] methods = activityClass.getDeclaredMethods();
	for (final Method method : methods) {
		OnClick onClickAnnotation = method.getAnnotation(OnClick.class);
		if (onClickAnnotation == null) {
			continue;
		}
		int viewResId = onClickAnnotation.value();
		if (viewResId == OnClick.NONE) {
			continue;
		}
		try {
			final View view = activity.findViewById(viewResId);
			if (view != null) {
				view.setOnClickListener(new View.OnClickListener() {
					@Override
					public void onClick(View v) {
						try {
							method.invoke(activity, view);
						} catch (Exception e) {
							e.printStackTrace();
						}
					}
				});
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

## 缺陷

以上的代码是有性能缺陷的，因为使用的是运行时注解，对性能有影响。如果使用编译时注解搭配APT来生成相关的代码，性能与手动写的代码没有区别。
编译时注解的具体实现看这篇文章——{% post_link 利用编译时注解实现一个ButterKnife 利用编译时注解实现一个ButterKnife %}

## 参考来源

* [Android 进阶 教你打造 Android 中的 IOC 框架 【ViewInject】 （上）](http://blog.csdn.net/lmj623565791/article/details/39269193)
* [Android 进阶 教你打造 Android 中的 IOC 框架 【ViewInject】 （下）](http://blog.csdn.net/lmj623565791/article/details/39275847)