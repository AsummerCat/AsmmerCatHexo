---
title: JVM笔记字符串常量池
date: 2020-07-22 14:54:58
tags: [JVM笔记]
---

# JVM笔记字符串常量池

## String的基本特性
![String的基本特性](/img/2020-07-02/9.png)  

### jdk1.9之后 String底层改变

```
jdk1.8 String底层:  char[] 构成 (字符数组 16个字节)

1.9之后 String底层: byte[] 构成 (字节数组 16个字节)

作用:节约空间
如果是中文->会有一个flag进行标记 使用两个byte

```
<!--more-->

#### 设置字符串常量池大小的JVM参数

```
-XX:StringTableSize=1009

在jdk1.6中默认是1009
jdk1.7中默认60013 
jdk1.8以后 如果手动设置的话 `1009是可设置的最小值`

字符串常量池底层是有个HashTable维护的
```


## 进入字符串常量池的方式
8种基本类型的常量池都是系统协调的，String类型的常量池比较特殊。它的主要使用方法有两种：
```
1.直接使用双引号声明出来的String对象会直接存储在常量池中。

2.如果不是用双引号声明的String对象，可以使用String提供的intern方法。
intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中
```

## Java中两种创建字符串对象的方式的分析。
### 双引号声明的String 分析
```
String s1 = "abc";
String s2 = "abc";
System.out.println(s1==s2);

返回 True

```
简单来说: 如果字符串常量池没有 (abc)该字符串  
则在字符串常量池创建 (abc)该字符串,后续的其他字符串引用则使用这个
然后将池中"abc"这个对象的引用地址返回给"abc"对象的引用s1

所以不管有多少个相同字符串都会引用一个字符串常量池的位置


以下的是 : 两个对象的引用
```
String s3 = new String("xyz");
String s4 = new String("xyz");
System.out.println(s3==s4);

返回false

s3 、s4是两个指向不同对象的引用，结果当然是false。
```

### intern()函数
```
池化
意思是: 将new String 对象 (如果字符串存在常量池中 则 无效果)移动到字符串常量池


注意如果字符串常量池里已经有了该字符串
调用intern()则无效果
例如:

String str=new String("1");
str.intern(); //这个不等于 常量池的"1"


String str=new String("1")+new String("2");
str.intern(); //这个等于 常量池的"12"
因为常量池里没有"12"这个字符串

注意2:
总之呢: 如果在intern()前 常量池已经存在该字符串 
那么执行intern()的对象则不相等于常量池 

```
![intern()注意](/img/2020-07-02/71.png)  

### 测试
##### 以下三种方式是相等的
```
String str="Hel" + "lo"; 
String str="Hello"; 
String str=new String("He")+new String("llo").intern();

```
##### 以下两种方式都不相等的
```
以下两种方式都不相等的
1. new对象
String str=new String("Hello");

因为这个是创建了一个对象,而不是引用了字符串常量池的内容

2. 存在变量拼接
String a="Hel";
String str=a + "lo"; 
因为变量拼接的底层是Stringbuilder
如果主动调用intern() 可以推送到字符串常量池


如果拼接的内容是(final)常量,则还是使用字符串常量池 


```
![测试](/img/2020-07-02/70.png)  

## 字符串常量池是有垃圾回收的