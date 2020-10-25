---
title: JVM参数详细
date: 2018-12-11 14:08:13
tags: jvm
---


堆设置 
-Xms=n 
初始堆大小。 
-Xmx=n 
最大堆大小。 
-Xmn=n 
<!--more-->
新生代大小，该配置优先于-XX:NewRatio，即如果配置了-Xmn，-XX:NewRatio会失效。 
-XX:NewRatio=n 
老生代和新生代的比值，例如该值为3，则表示新生代与老生代比值为1:3。 
-XX:SurvivorRatio=n 
新生代中 Eden区和Survivor区的比值。Survivor区分为等价的两个区S0，S1。例如-XX:SurvivorRatio=3，则表示Eden:S0:S1=3:1:1。 
-XX:PermSize=n 
设置永久代（方法区）大小，Java 8之后被移除。 
-XX:MaxPermSize=n 
设置永久代（方法区）最大大小，Java 8之后被移除。 
-XX:MetaspaceSize=n 
设置本地元空间Metaspace大小，Java 8以后移除了方法区，取而代之的是本地元空间Metaspace。 
-XX:MaxMetaspaceSize=n 
设置本地元空间Metaspace最大大小，Java 8以后移除了方法区，取而代之的是本地元空间Metaspace。 
-XX:MaxGCPauseMillis=n 
设置垃圾收集最大暂停时间。 
-XX:GCTimeRatio=n 
设置一次垃圾回收时间占程序运行时间的百分比，花费在GC上的时间比例不超过1 / (1 + n)。

垃圾收集器设置 
-XX:+UseSerialGC 
设置串行收集器。 
-XX:+UseParallelGC 
设置并行收集器。 
-XX:+UseParallelOldGC 
设置并行老年代收集器。 
-XX:+UseParNewGC 
设置并行新生代收集器，可以和老年代的CMS共用。 
-XX:+UseConcMarkSweepGC 
设置并发收集器。 
-XX:+UseG1GC 
G1垃圾回收器。

并行收集器设置 
-XX:ParallelGCThreads=n 
设置并发收集器年轻代收集方式为并行收集时的线程数。

并发收集器设置 
-XX:+CMSIncrementalMode 
设置为增量模式。在增量模式下，并发收集器在并发阶段，不会独占整个周期，而会周期性的暂停，唤醒应用线程。收集器把并发阶段工作划分为片段，安排在次级(minor) 回收运行。适用于需要低延迟，CPU少的服务器。一般单CPU会开启增量模式。 
-XX:ParallelCMSThreads=n 
并发垃圾回收器标记扫描所使用的线程数量

垃圾回收统计信息 
-XX:+PrintGC 
输出GC日志，简要日志。 
-XX:+PrintGCDetails 
打印GC日志，详细展示每个区的日志。 
-XX:+PrintGCTimeStamps 
打印GC日志，详细展示GC运行时间的日志。输出GC的时间戳以基准时间的形式展示。 
-XX:+PrintGCDateStamps 
打印GC日志，详细展示GC运行时间的日志。输出GC的时间戳以以日期的形式展示。 
-XX:+PrintHeapAtGC 
打印在进行GC的前后打印出堆的信息。 
-Xloggc:../logs/gc.log 
日志文件的输出路径。

