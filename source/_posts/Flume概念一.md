---
title: Flume概念一
date: 2022-08-12 09:39:24
tags: [Flume,大数据]
---
# Flume概念
Flume是Cloudera提供的一个高可用的,高可靠的,分布式的海量日志采集,聚合和传输的系统;
Flume基于流式架构,灵活简单.
只支持文本数据;

<!--more-->


# Flume架构
![image](/img/2022-08-01/Flume.png)

### Agent
Agent是一个JVM进程,它以事件的形式将数据从源头送至目的地.
Agent主要有三个部分组成,<font color='red'>Source ,Channel,Sink</font>

####  Source  (数据源)
Source是福州接收数据到Flume Agent的组件;
Source组件可以处理各种类型,各种格式的日志数据,包括<font color='red'>
```
avro, thrift, exec, jms,
spooling directory, netcat, 
taildir, sequence generator, 
syslog, http, legacy.
```
</font>

#### Sink (输出源)
Sink不断轮询Channel中的事件且批量的移除它们,并将这些事件批量写入到存储或索引系统,或者被发送到另一个Flume Agent;
Sink组件目的地包括<font color='red'>
```
hdfs , logger , avro , thrift ,
ipc , file , Hbase , solr , 自定义
```
</font>

#### Channel (缓存区)
Channel是位于Source和Sink之间的缓冲区,因此,Channel允许Source和Sink运作在不同的速率上.
channel是线程安全的,可以同时处理几个Source的写入操作和几个Sink的读取操作;

Flume自带两种Channel: `Memory Channel`和`File Channel`;

`Memory Channel`是内存中的队列,在不关系数据丢失的情景下适用;

`File Channel`会将所有事件写入到磁盘,因此在程序关闭或者宕机的时候不会丢失数据;

## Event
传输单元,Flume数据传输的基本单元,以Event的形式将数据从源头送至目的地;
Event由<font color='red'>Header</font><font color='red'>Body</font>两部分组成,Header用来存放该event的一些属性,为K-V结构,Body用来存放该条数据,形式为字节数组;

 
 