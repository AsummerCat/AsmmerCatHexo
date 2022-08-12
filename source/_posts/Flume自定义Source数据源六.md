---
title: Flume自定义Source数据源六
date: 2022-08-12 09:44:38
tags: [Flume,大数据]
---
# Flume自定义Source数据源

```
Source 是负责接收数据到 Flume Agent 的组件。Source 组件可以处理各种类型、各种
格式的日志数据，包括 avro、thrift、exec、jms、spooling directory、netcat、sequence 
generator、syslog、http、legacy。官方提供的 source 类型已经很多，但是有时候并不能
满足实际开发当中的需求，此时我们就需要根据实际需求自定义某些 source。
官方也提供了自定义 source 的接口：
https://flume.apache.org/FlumeDeveloperGuide.html#source
```
<!--more-->

根据官方说明自定义
MySource 需要继承 `AbstractSource` 类并实现 `Configurable` 和 `PollableSource` 接口。
实现相应方法：
```
getBackOffSleepIncrement() //backoff 步长
getMaxBackOffSleepInterval()//backoff 最长时间
configure(Context context)//初始化 context（读取配置文件内容）
process()//获取数据封装成 event 并写入 channel，这个方法将被循环调用。
```

使用场景：读取 MySQL 数据或者其他文件系统。

## 导入依赖
```
<!-- flume核心依赖 -->
 <dependency>
     <groupId>org.apache.flume</groupId>
     <artifactId>flume-ng-core</artifactId>
     <version>1.9.0</version>
 </dependency>
```

## 实现
```
import java.util.HashMap;
public class MySource extends AbstractSource implements 
Configurable, PollableSource {
 //定义配置文件将来要读取的字段
 private Long delay;
 private String field;
 //初始化配置信息
 @Override
 public void configure(Context context) {
 delay = context.getLong("delay");
 field = context.getString("field", "Hello!");
 }
 @Override
 public Status process() throws EventDeliveryException {
 try {
 //创建事件头信息
 HashMap<String, String> hearderMap = new HashMap<>();
 //创建事件
 SimpleEvent event = new SimpleEvent();
 //循环封装事件
 for (int i = 0; i < 5; i++) {
 //给事件设置头信息
 event.setHeaders(hearderMap);
 //给事件设置内容
 event.setBody((field + i).getBytes());
 //将事件写入 channel
 getChannelProcessor().processEvent(event);
 Thread.sleep(delay);
 }
 } catch (Exception e) {
 e.printStackTrace();

 return Status.BACKOFF;
 }
 return Status.READY;
 }
 @Override
 public long getBackOffSleepIncrement() {
 return 0;
 }
 @Override
 public long getMaxBackOffSleepInterval() {
 return 0;
 } }
```

## 打包成jar
将写好的代码打包，并放到 flume 的 lib 目录（/opt/module/flume）下。

## flume启动配置文件
```
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 自定义source数据源 全类名
a1.sources.r1.type = com.atguigu.MySource
# 自定义配置文件信息delay,field 在configure()中
a1.sources.r1.delay = 1000
#a1.sources.r1.field = atguigu

a1.sinks.k1.type = logger

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```