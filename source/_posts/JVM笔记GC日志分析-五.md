---
title: JVM笔记GC日志分析(五)
date: 2020-07-25 15:13:34
tags: [JVM笔记]
---

# JVM笔记GC日志分析(五)

## 内存分配和垃圾回收的参数列表
```
1.-XX:+PrintGC  
输出GC日志

2.-XX:+PrintGCDetails
输出GC的详细日志

3.-XX:+PrintGCTimeStamps
输出GC的时间戳(以基准时间的形式)

4.-XX:+PrintGCDateStamps 
输出GC的时间戳(以日期的形式,如2013-05-01)

5.-XX:PrintHeapAtGC
在进行GC的前后打印出堆的信息

6.-Xloggc:../logs/gc.log
日志文件的输出路径
```
<!--more-->

![日志分析](/img/2020-07-02/109.png) 
##### 补充说明
![日志分析](/img/2020-07-02/110.png)   
![日志分析](/img/2020-07-02/111.png)   
![日志分析](/img/2020-07-02/112.png)   
![日志分析](/img/2020-07-02/113.png)   


## 日志分析的使用

#### 常用工具
```
常用工具:
  1.GCViewer
  
  2.GCEasy      推荐(浏览器 gceasy.io 格式好看)
 
  3.GCHisto
 
  4.GCLogViewer
  
  5.Hpjmeter
 
  6.garbagecat等
  
  首先导出日志
  -Xloggc:/path/to/gc.log
```