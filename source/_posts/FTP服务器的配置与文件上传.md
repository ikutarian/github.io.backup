---
title: FTP服务器的配置与文件上传
date: 2018-10-26 09:33:03
tags:
  - FTP
  - 文件上传
categories:
  - 安装与配置
---

最近在开发时，遇到了搭建 FTP 服务器，并利用 Java 代码上传文件的需求，现在总结一下

<!-- more -->

## FTP 服务器的安装与配置

FTP 服务器在 Linux 与 Windows 平台下都有。Linux 下可以使用 vsftpd，Windows 下可以使用 Filezilla Server。目前我的开发平台是 Windows，所以使用的是 Filezilla Server

### 安装

打开[官网](https://filezilla-project.org/download.php?type=server)，下载 exe 安装包，直接安装即可

### 配置

进入 Filezilla Server Interface 的界面之后，首先要让 FTP 服务器上线，点亮这个闪电图标即可

{% asset_img Snipaste_2018-10-26_09-41-02.png %}

然后就是增加一个用户和给用户分配一个目录。点击这个图标，进入用户管理

{% asset_img Snipaste_2018-10-26_09-45-10.png %}

点击 Add

{% asset_img Snipaste_2018-10-26_09-46-24.png %}

输入用户名，并点击“OK”

{% asset_img Snipaste_2018-10-26_09-47-17.png %}

设置密码

{% asset_img Snipaste_2018-10-26_09-48-27.png %}

切换到 Shared folders 标签页，给新增的用户指定一个目录

{% asset_img Snipaste_2018-10-26_09-49-53.png %}

并指定对文件夹的操作权限

{% asset_img Snipaste_2018-10-26_09-51-00.png %}

这样就好了，点击“OK”即可

{% asset_img Snipaste_2018-10-26_09-52-05.png %}

### 验证

打开浏览器输入 ftp://127.0.0.1，然后再输入用户名和密码即可

{% asset_img Snipaste_2018-10-26_09-55-21.png %}

## 上传文件

引入依赖

```xml
 <dependency>
    <groupId>commons-net</groupId>
    <artifactId>commons-net</artifactId>
    <version>3.6</version>
</dependency>
```

然后是 Java 代码

```java
import com.ikutarian.mmall.common.Config;
import org.apache.commons.net.ftp.FTPClient;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.List;

public class FtpUtils {

    private static final Logger log = LoggerFactory.getLogger(FtpUtils.class);

    public static boolean uploadFile(List<File> fileList) throws IOException {
        log.info("开始连接FTP服务器，上传图片");
        boolean result = uploadFile(Config.getImgBasePath(), fileList);
        log.info("结束上传，上传结果: {}", result);
        return result;
    }

    private static boolean uploadFile(String remotePath, List<File> fileList) throws IOException {
        FTPClient ftpClient = new FTPClient();

        try {
            ftpClient.connect(Config.getFtpIp(), Config.getFtpPort());
            ftpClient.login(Config.getFtpUsername(), Config.getFtpPassword());
        } catch (IOException e) {
            log.error("连接FTP服务器异常", e);
            return false;
        }

        try {
            ftpClient.makeDirectory(remotePath);  // 如果remotePath不存在，就创建一个，当然首先要有权限才行
            ftpClient.changeWorkingDirectory(remotePath);
            ftpClient.setBufferSize(1024);
            ftpClient.setControlEncoding("UTF-8");
            ftpClient.setFileType(FTPClient.BINARY_FILE_TYPE);
            ftpClient.enterLocalPassiveMode();
            for (File file : fileList) {
                FileInputStream fis = new FileInputStream(file);
                ftpClient.storeFile(file.getName(), fis);
                fis.close();
            }
        } catch (IOException e) {
            log.error("文件上传异常", e);
            return false;
        } finally {
            ftpClient.disconnect();
        }

        return true;
    }
}
```

`Config` 类是我的配置类，配置 FTP 的连接信息，配置信息写在 properties 文件中

```
# 图片存放的根路径
img.basePath=img

# FTP
ftp.ip=127.0.0.1
ftp.port=21
ftp.user=ikutarian
ftp.password=123456
```

其中有一句代码很重要 `ftpClient.enterLocalPassiveMode();`。这个涉及到了 FTP 服务器的主动模式与被动模式的知识。下面来分别说明一下

```
By default, the FTP protocol establishes a data connection by opening a port on the client and allows the server connecting to this port. This is called local active mode, but it is usually blocked by firewall so the file transfer may not work. Fortunately, the FTP protocol has another mode, local passive mode, in which a data connection is made by opening a port on the server for the client to connect – and this is not blocked by firewall.

So it is recommended to switch to local passive mode before transferring data, by invoking the methodenter LocalPassiveMode() of the FTPClient class.
```

翻译一下：通常 FTP 协议会在客户端上开启一个端口，让服务器连接到这个端口。这就是 `local active mode`（主动模式）。但是通常由于防火墙的存在，没办法传输数据。

{% asset_img 20171226094239038.jpg %}

FPT 协议还有另外一个模式：`local passive mode`（被动模式）。在服务器上开启一个端口，让客户端连接到这个端口，这样就不会被防火墙拦截了。所以，建议在传输数据之前，切换到 `local passive mode` 模式

{% asset_img 20171226094256663.jpg %}