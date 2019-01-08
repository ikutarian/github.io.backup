---
title: MyBatis三剑客
date: 2018-10-20 13:22:56
tags:
  - MyBatis
  - MyBatis-Generator
  - MyBatis-Plugin
  - MyBatis-PageHelper
categories:
  - 安装与配置
---

使用 Mybatis 时，为了提高效率，会用到三个工具

- MyBatis-Generator
- MyBatis-Plugin
- MyBatis-PageHelper

<!-- more -->

## MyBatis-Generator

用来生成 pojo、dao 和 mapper 文件的工具

### 官方文档

http://www.mybatis.org/generator/quickstart.html

### 配置

1. 配置数据连接信息

```xml
<!-- 数据库驱动、URL、用户名、密码 -->
<jdbcConnection driverClass="com.mysql.jdbc.Driver"
                connectionURL="jdbc:mysql://127.0.0.1:3306/mmall?characterEncoding=utf-8"
                userId="root"
                password="">
</jdbcConnection>
```

2. 生成模型的包名和位置

```xml
<!-- 生成模型的包名和位置 -->
<javaModelGenerator targetPackage="com.ikutarian.mmall.pojo" targetProject="src">
    <property name="enableSubPackages" value="true" />
    <property name="trimStrings" value="true" />
</javaModelGenerator>
```

3. 生成的映射文件包名和位置

```xml
<!-- 生成的映射文件包名和位置 -->
<sqlMapGenerator targetPackage="com.ikutarian.mmall.mapper" targetProject="src">
   <property name="enableSubPackages" value="true" />
</sqlMapGenerator>
```

4. 生成DAO的包名和位置

```xml
<!-- 生成DAO的包名和位置 -->
<javaClientGenerator type="XMLMAPPER" targetPackage="com.ikutarian.mmall.dao"  targetProject="src">
   <property name="enableSubPackages" value="true" />
</javaClientGenerator>
```

5. 表与JavaBean

只需要更改 `tableName` 和 `domainObjectName` 就可以

对于普通的表

```xml
<table tableName="mmall_shipping" domainObjectName="Shipping" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
```

对于有些字段类型为 `TEXT` 的表，需要使用 `columnOverride` 标签重新指定一下，否则不会为该字段生成 JavaBean 的属性

```xml
<table tableName="mmall_product" domainObjectName="Product" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" >
   <columnOverride column="detail" jdbcType="VARCHAR"/>
   <columnOverride column="sub_images" jdbcType="VARCHAR"/>
</table>
```

### 使用

在控制台中输入

```
java -jar mybatis-generator-core-1.3.7.jar -configfile generatorConfig.xml -overwrite
```

一个完整的配置文件例子，可以在 [generatorConfig.xml](https://github.com/ikutarian/mmall/blob/master/mybatis-generator/generatorConfig.xml) 看到


## MyBatis-Plugin

下载一个 IDEA 的 plugin 安装即可，如下图

{% asset_img Snipaste_2018-10-20_13-32-50.png %}

但是因为 Mybatis-Plugin 现在收费了，可以下载 “Free Mybatis Plugin”

{% asset_img QQ截图20181022134806.png %}

打开一个 DAO 文件，就可以看到左边出现了很多箭头

{% asset_img Snipaste_2018-10-20_14-11-26.png %}

点击箭头，就可以跳转到相应的 mapper 的 xml 配置

{% asset_img Snipaste_2018-10-20_14-12-40.png %}

## MyBatis-PageHelper

官网：https://github.com/pagehelper/Mybatis-PageHelper

现在最新版本是 5.1.7 了，配置和 4.x 时不太一样

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="mapperLocations" value="classpath:mapper/*Mapper.xml"/>

    <!-- Mybatis-PageHelper -->
    <property name="plugins">
        <array>
            <bean class="com.github.pagehelper.PageInterceptor">
                <property name="properties">
                    <value>
                        helperDialect=mysql
                    </value>
                </property>
            </bean>
        </array>
    </property>
</bean>
```