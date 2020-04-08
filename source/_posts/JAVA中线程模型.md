---
title: JAVA中线程模型
date: 2020-04-08 21:33:55
tags: [java,多线程]
---

# JAVA中线程模型

底层是调用本地方法实现的

## 调用过程底层


```
java创建一个Thread对象,执行start()
->native的start0方法 
->JVM hotspot 的newJavaThread方法 
->调用C++的 pthread_create() 方法  在linux上创建了线程, 回调java的run方法
(run方法这个是pthread_create方法中的第三个参数 )

这样目的是让 JVM管理监控线程 而不是直接交给OS去操作

java线程数 和OS线程数 1对1

```