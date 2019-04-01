---
title: Web工程下读取文件的几种方法
date: 2019-04-01 20:30:37
tags: [java]
---

# Web工程下读取文件的几种方法

## **读取文件系统路径文件 \* 一定要使用绝对路径**

```
// 读取WebRoot下
String fileName = getServletContext().getRealPath("/index.jsp"); 

// 读取WebRoot/WEB-INF/a.properties
String fileName2 = getServletContext().getRealPath(
"/WEB-INF/a.properties");
```

## **读取classpath下文件**

```
// 读取src下info.txt
String fileName3 = ReadFile.class.getResource("/info.txt").getFile();

// 读取 src下cn.itcast.config包下info.txt
String fileName4 = ReadFile.class.getResource(
"/cn/itcast/config/info.txt").getFile();
```

<!--more-->

## **使用ResourceBundle 快速读取src下properties文件**

```
// 读取src下 基名为myproperties的properties文件，获取其中name配置值

String value = ResourceBundle.getBundle("myproperties").getString(
"name");
```

## **使用Properties类加载Properties文件**

```
/ 读取src下b.properties
InputStream in = ReadFile.class.getResourceAsStream("/b.properties");
Properties properties = new Properties();
properties.load(in);
String value2 = properties.getProperty("name"); // 获得name属性
```



## 当前class的位置的路径 

```
String uploadPath = this.getClass().getClassLoader().getResource("").getPath();
```

