---
title: JDK9模块化输入输出
date: 2022-02-09 22:27:52
tags: [java,jdk新特性]
---

# JDK9模块化输入输出

使用场景: 
比如多模工程中 common模块引用biz模块内容 
可指定biz对外输出功能 减小jar大小

## 创建输出模块
常规创建类 之后  
新增 idea :new ->module-info.java  
创建一个模块信息定义文件

tips: 模块名称 可以用 com.xx.xx 来定义
<!--more-->
```
moodule 模块名称A {
    exports com.xx.需要输出的包名;
    
}
```


## 创建输入模块
B工程需要使用的话引入A创建的输出模块

注意需要先添加依赖,不然模块A会报错
```
module B{
    requires 模块名称A ;
}

```

## 引用输出
