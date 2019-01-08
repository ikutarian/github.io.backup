---
title: 使用Bugly应用升级SDK实现App更新
date: 2018-09-17 10:51:57
tags:
  - Bugly
  - 更新
  - SDK
categories:
  - Android
---

## Android Studio 配置

### 依赖配置

在 `app/build.gradle` 下加入依赖

```
dependencies {
    compile 'com.tencent.bugly:crashreport_upgrade:latest.release'1.2.0
    compile 'com.tencent.bugly:nativecrashreport:latest.release'
}
```

<!-- more -->

### 参数配置

#### 1. 权限配置

在 `AndroidMainfest.xml` 中进行以下配置：

```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_LOGS" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

#### 2. Activity 配置

```xml
<activity
    android:name="com.tencent.bugly.beta.ui.BetaActivity"
    android:configChanges="keyboardHidden|orientation|screenSize|locale"
    android:theme="@android:style/Theme.Translucent" />
```

#### 3. 配置 FileProvider

如果您想兼容 Android N 或者以上的设备，必须要在 `AndroidManifest.xml` 文件中配置 `FileProvider` 来访问共享路径的文件

```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="${applicationId}.fileProvider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"/>
</provider>
```

在 `res` 目录新建 `xml` 文件夹，创建 `provider_paths.xml` 文件，文件内容如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- /storage/emulated/0/Download/${applicationId}/.beta/apk-->
    <external-path name="beta_external_path" path="Download/"/>
    <!--/storage/emulated/0/Android/data/${applicationId}/files/apk/-->
    <external-path name="beta_external_files_path" path="Android/data/"/>
</paths>
```

#### 4. 混淆配置

```
-dontwarn com.tencent.bugly.**
-keep public class com.tencent.bugly.**{*;}
-keep class android.support.**{*;}
```

## SDK的使用

### 封装一个工具类

为了维护方便，我封装了一个工具类，在工具类中统一管理 Bugly 的 API

```java
public class BuglyUtil {

    /**
     * 初始化SDK
     */
    public static void init(Context context) {
        // true表示初始化时自动检查升级; false表示不会自动检查升级,需要手动调用Beta.checkUpgrade()方法;
        Beta.autoCheckUpgrade = false;
        // 只允许在MainActivity上显示更新弹窗，其他activity上不显示弹窗
        Beta.canShowUpgradeActs.add(MainActivity.class);
        Bugly.init(context, "注册时申请的APPID", false);
    }

    /**
     * 静默检查更新，并弹窗
     */
    public static void checkUpdate() {
        /**
         * @param isManual  用户手动点击检查，非用户点击操作请传false
         * @param isSilence 是否显示弹窗等交互，[true:没有弹窗和toast] [false:有弹窗或toast]
         */
        Beta.checkUpgrade(false, false);
    }
}
```

关于 `Beta` 类的 API，可以查看[Bugly Android 应用升级 SDK 高级配置](https://bugly.qq.com/docs/user-guide/advance-features-android-beta/?v=20170912151050)

## 使用

在 `Application` 中初始化 SDK

```java
public class AppApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();

        BuglyUtil.init(this);
    }
}
```

在 `MainActivity` 中检查更新

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        BuglyUtil.checkUpdate();
    }
}
```

## 参考

* [官方文档](https://bugly.qq.com/docs/user-guide/instruction-manual-android-upgrade/?v=20170912151050)
* [Bugly Android 应用升级 SDK 高级配置](https://bugly.qq.com/docs/user-guide/advance-features-android-beta/?v=20170912151050)
*  [Bugly实现app全量更新](http://blog.csdn.net/qq_33689414/article/details/54911895)