---
title: 图片合成、拼接与Base64转换
date: 2018-10-23 17:10:15
tags:
  - 图片处理
  - Base64
categories:
  - Java
---

在最近的开发中，遇到了图片拼接、合成与Base64编码转换的问题，现在进行总结

<!-- more -->

## 图片拼接

```java
/**
 * 纵向拼接多张图片
 * 宽度必须一样，否则会抛异常
 *
 * @param imgs    图片地址集合
 * @param type    图片类型
 * @param dst_pic 输出的文件的路径：F:/test2.jpg
*/
public static boolean splice(String[] imgs, String type, String dst_pic) {
    //获取需要拼接的图片长度
    int len = imgs.length;
    //判断长度是否大于0
    if (len < 1) {
        return false;
    }
    File[] src = new File[len];
    BufferedImage[] images = new BufferedImage[len];
    int[][] ImageArrays = new int[len][];
    for (int i = 0; i < len; i++) {
        try {
            src[i] = new File(imgs[i]);
            images[i] = ImageIO.read(src[i]);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        int width = images[i].getWidth();
        int height = images[i].getHeight();
        // 从图片中读取RGB 像素
        ImageArrays[i] = new int[width * height];
        ImageArrays[i] = images[i].getRGB(0, 0, width, height, ImageArrays[i], 0, width);
    }

    int dst_height = 0;
    int dst_width = images[0].getWidth();
    //合成图片像素
    for (int i = 0; i < images.length; i++) {
        dst_width = dst_width > images[i].getWidth() ? dst_width : images[i].getWidth();
        dst_height += images[i].getHeight();
    }
    //合成后的图片
    System.out.println("宽度:" + dst_width);
    System.out.println("高度:" + dst_height);
    if (dst_height < 1) {
        System.out.println("dst_height < 1");
        return false;
    }
    // 生成新图片
    try {
        dst_width = images[0].getWidth();
        BufferedImage ImageNew = new BufferedImage(dst_width, dst_height,
                BufferedImage.TYPE_INT_RGB);
        int height_i = 0;
        for (int i = 0; i < images.length; i++) {
            ImageNew.setRGB(0, height_i, dst_width, images[i].getHeight(),
                    ImageArrays[i], 0, dst_width);
            height_i += images[i].getHeight();
        }

        File outFile = new File(dst_pic);
        ImageIO.write(ImageNew, type, outFile);// 写图片 ，输出到硬盘
    } catch (Exception e) {
        e.printStackTrace();
        return false;
    }
    return true;
}
```

## 合成

```java
/**
 * 两张图片合成一张图片
 *
 * @param backImage 背景图文件地址
 * @param srcImage  前景图文件地址
 * @param descImage 生成图文件地址
 */
public static void merge(String backImage, String srcImage, String descImage) {
    try {
        BufferedImage backBufferedImage = ImageIO.read(new File(backImage));
        BufferedImage srcBufferedImage = ImageIO.read(new File(srcImage));
        // 输出图片宽度
        int width = backBufferedImage.getWidth();
        // 输出图片高度
        int height = backBufferedImage.getHeight();
        BufferedImage descBufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_4BYTE_ABGR);
        Graphics2D graphics2d = (Graphics2D) descBufferedImage.getGraphics();
        graphics2d.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
        // 往画布上添加图片,并设置边距
        graphics2d.drawImage(backBufferedImage, null, 0, 0);
        graphics2d.drawImage(srcBufferedImage, null, 0, 0);
        graphics2d.dispose();
        // 输出新图片
        ImageIO.write(descBufferedImage, "png", new File(descImage));
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

## Base64

一张图片

{% asset_img demo.jpg %}

转换成 Base64 之后，得到

```
data:image/jpeg;base64,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
```

```java
/**
 * 将base64编码字符串转换为图片
 *
 * @param base64Code 图片的Base64编码
 * @param outputPath 输出文件的路径
 * @return 转换是否成功 true-成功 false-失败
 */
public static boolean generateImage(String base64Code, String outputPath) {
    if (base64Code == null) {
        return false;
    }

    base64Code = cutBase64Header(base64Code);
    BASE64Decoder decoder = new BASE64Decoder();
    try {
        // 解密
        byte[] b = decoder.decodeBuffer(base64Code);
        // 处理数据
        for (int i = 0; i < b.length; ++i) {
            if (b[i] < 0) {
                b[i] += 256;
            }
        }
        OutputStream out = new FileOutputStream(outputPath);
        out.write(b);
        out.flush();
        out.close();
        return true;
    } catch (Exception e) {
        return false;
    }
}

private static String cutBase64Header(String base64Code) {
    // 这一步很重要很重要，因为base64的数据会有 data:image/png;base64,
    // 所以需要将这个头截取掉之后再转换成图片，否则图片就打不开
    return base64Code.substring(base64Code.indexOf(",") + 1);
}
```

使用

```java
String base64 = "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQEAAQABAAD/2wBDAAUDBAQEAwU...";
generateImage(base64, "f:/base64_result.jpg");
```

## 使用开源库

关于图片处理，有一些开源库

- 阿里巴巴的 SimpleImage
- Google的 thumbnailator