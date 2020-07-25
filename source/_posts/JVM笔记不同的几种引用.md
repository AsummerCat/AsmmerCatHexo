---
title: JVM笔记不同的几种引用
date: 2020-07-25 15:02:42
tags: [JVM笔记]
---

# JVM笔记不同的几种引用

## 引用

![引用](/img/2020-07-02/92.png)  

### 强引用
`Strong Reference`  
不回收   

->我们new的对象基本都是强引用

`强引用是内存泄露的主要原型之一`
<!--more-->

### 软引用
`Soft Reference`  
内存不足即回收  

通常: 用来实现内存敏感的缓存  
例如mybatis里大量使用了软引用


### 弱引用 
`Weak Reference`  
GC时候有`弱引用`就会被回收   
例如:  
`WeakHashMap`



### 虚引用
`Phantom Reference`  
对象回收跟踪    

唯一作用就是:  
为一个对象设置一个`虚引用`,在被收集器回收时获取一个系统通知