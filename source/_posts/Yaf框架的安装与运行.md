---
title: Yaf框架的安装与运行
date: 2019-01-08 15:14:54
tags:
  - Yaf
  - 框架
  - PHP
categories:
  - PHP
---

Yaf 是鸟哥开发的一个高性能框架，相对于原生的 PHP，性能只降低不到 10%

<!-- more -->

# 确认 PHP 的版本

查看 `phpinfo.php`

{% asset_img QQ截图20190108110922.png %}

可以看到：

- Version: 7.2.9
- Compiler: MSVC15 (Visual C++ 2017)
- Architecture: x86
- Thread Safety: enabled

# Windows下的安装

总的来说就两步：

1. 下载 DLL 文件，放到 `php/ext` 文件夹
2. 配置 `php.ini` 文件

## 下载

打开[网站](https://pecl.php.net/package/yaf)，选择最新的版本，而且我是 Windows 平台，点击 `DLL` 这个图标

{% asset_img QQ截图20190108110807.png %}

根据 PHP 的版本（7.2.9、Thread Safe、X86），选择这个版本的 yaf 下载

{% asset_img QQ截图20190108140754.png %}

下载之后，打开压缩包，把 `php_yaf.dll` 文件放到 `php\ext` 目录下

## 配置

打开 `php.ini`，在最后加入以下配置信息

```ini
extension=php_yaf.dll
```

完成以上配置后，重启 Apache，在 `phpinfo.php` 中就能看到 yaf 的配置了

{% asset_img QQ截图20190108111536.png %}

# 运行

打开 [Yaf 的 Github](https://github.com/laruence/yaf/releases)，下载一个最新版本的 release 的压缩包

打开压缩包，进入 `tools/cg` 目录，执行以下命令

```
php yaf_cg sample
```

`sample` 是项目的名称，可以自定义。命令执行完毕输出 `DONE` 之后，可以在当前目录下看到一个 `out` 文件夹，进入就可以看到一个名为 `sample` 的文件夹了

把 `sample` 复制到 `htdocs` 中，然后在浏览器中访问 http://localhost/sample/ 就能看到界面了

{% asset_img QQ截图20190108142534.png %}
