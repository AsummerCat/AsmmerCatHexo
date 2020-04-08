---
title: JVM实战
date: 2020-04-08 21:31:47
tags: [虚拟机调优,JVM]
---

# JVM调优参数




```
jvm启动参数时强烈建议加入如下的一些参数：

-XX:+HeapDumpOnOutOfMemoryError

-XX:HeapDumpPath=/usr/local/app/oom

加入了这两个参数，在jvm OOM崩溃时，你能够去找到OOM时的内存快照
```


```
jvm参数的标准模板如下：

-Xms4096M -Xmx4096M -Xmn3072M -Xss1M
-XX:MetaspaceSize=256M 
-XX:MaxMetaspaceSize=256M
-XX:+UseParNewGC
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFaction=92
-XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction=0
-XX:+CMSParallelInitialMarkEnabled
-XX:+CMSScavengeBeforeRemark
-XX:+DisableExplicitGC -XX:+PrintGCDetails
-Xloggc:gc.log -XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/usr/local/app/oom
```

<!--more-->

## 我们写的Java代码到底是如何运行起来的？

1、我们编写.java文件

2、打包成.class文件

3、JVM类加载器把.class字节码文件加载到内存中

4、JVM基于自己的字节码执行引擎，来执行加载到内存里我们写好的类

## JVM类加载机制
一个类从加载到使用，一般会经历下面的这个过程：

加载-->验证-->准备-->解析-->初始化-->使用-->卸载

验证：校验加载进来的.class文件是否符合规范

准备：分配内存空间，分配变量类型对应的初始值（如int分配0，Integer分配null）

解析：把符号引替换为直接引用

初始化：初始我们分配的值（如int a = 3，那么这时a就初始为3）

## 使用 jstat 摸清线上系统的JVM运行状况
jps能找出Java进程的pid

jstat -gc PID可以看到这个Java进程的内存和GC情况了

## CPU负载过高的原因
1. 系统创建了大量线程
2. JVM在运行时频繁的full gc


## 频繁full gc的原因
1. 内存分配不合理
2. 存在内存泄漏问题
3. 永久代里的类太多，触发了full gc
4. 工程师错误的执行System.gc

## 导致对象进入老年代的原因
1. 对象在年轻代躲过15次垃圾回收
2. 对象太大，直接进入老年代
3. young gc后存活对象survivor放不下
4. 存活对象年龄超过survivor的50%


## 什么是内存溢出？在哪些区域会发生内存溢出？

1. 永久代（从jdk1.8后叫做Metaspace)用来存放你系统里的各种类的信息，此区域可以会发生内存溢出，即OOM。

2. 每个线程的虚拟机栈内存也可能OOM。

3. 堆内存空间会OOM。