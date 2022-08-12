---
title: Flume自定义sink七
date: 2022-08-12 09:45:05
tags: [Flume,大数据]
---
# Flume自定义sink

自定义输出源
[官方文档](https://flume.apache.org/FlumeDeveloperGuide.html#sink)

根据官方说明自定义
`MySink` 需要继承 `AbstractSink` 类并实现 `Configurable` 接口。
实现相应方法：
```
configure(Context context)//初始化 context（读取配置文件内容）
process()//从 Channel 读取获取数据（event），这个方法将被循环调用。
使用场景：读取 Channel 数据写入 MySQL 或者其他文件系统。
```
<!--more-->

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
使用 flume 接收数据，并在 Sink 端给每条数据添加前缀和后缀，输出到控制台。前后
缀可在 flume 任务配置文件中配置。
```
public class MySink extends AbstractSink implements Configurable 
{
 //创建 Logger 对象
 private static final Logger LOG = 
LoggerFactory.getLogger(AbstractSink.class);
 private String prefix;
 private String suffix;
 @Override
 public Status process() throws EventDeliveryException {
 //声明返回值状态信息
 Status status;
 //获取当前 Sink 绑定的 Channel
 Channel ch = getChannel();
 //获取事务
 Transaction txn = ch.getTransaction();
 //声明事件
 Event event;
 //开启事务
 txn.begin();

 //读取 Channel 中的事件，直到读取到事件结束循环
 while (true) {
 event = ch.take();
 if (event != null) {
 break;
 }
 }
 try {
 //处理事件（打印）
 LOG.info(prefix + new String(event.getBody()) + 
suffix);
 //事务提交
 txn.commit();
 status = Status.READY;
 } catch (Exception e) {
 //遇到异常，事务回滚
 txn.rollback();
 status = Status.BACKOFF;
 } finally {
 //关闭事务
 txn.close();
 }
 return status;
 }
 @Override
 public void configure(Context context) {
 //读取配置文件内容，有默认值
 prefix = context.getString("prefix", "hello:");
 //读取配置文件内容，无默认值
 suffix = context.getString("suffix");
 } }
```


## 打包成jar
将写好的代码打包，并放到 flume 的 lib 目录（/opt/module/flume）下。

## flume启动配置文件
```
#自定义sink 全类名
a1.sinks.k1.type = com.atguigu.MySink
# 自定义前缀 根据configure中进行配置
#a1.sinks.k1.prefix = atguigu:
# 自定义后缀
a1.sinks.k1.suffix = :atguigu

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```