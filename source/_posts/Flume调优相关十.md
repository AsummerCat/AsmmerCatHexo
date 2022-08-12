---
title: Flume调优相关十
date: 2022-08-12 09:46:50
tags: [Flume,大数据]
---
# Flume调优相关
## 你是如何实现 Flume 数据传输的监控的
使用第三方框架 Ganglia 实时监控 Flume。


##  Flume 的 Source，Sink，Channel 的作用？你们 Source 是什么类
型？
1）作用 （1）Source 组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，
包括 avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、
http、legacy
（2）Channel 组件对采集到的数据进行缓存，可以存放在 Memory 或 File 中。
（3）Sink 组件是用于把数据发送到目的地的组件，目的地包括 Hdfs、Logger、avro、
thrift、ipc、file、Hbase、solr、自定义。

<!--more-->

## Flume 参数调优
#### 1）Source
增加 Source 个（使用 Tair Dir Source 时可增加 FileGroups 个数）可以增大 Source
的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个
文件目录，同时配置好多个 Source 以保证 Source 有足够的能力获取到新产生的数据。
batchSize 参数决定 Source 一次批量运输到 Channel 的 event 条数，适当调大这个参
数可以提高 Source 搬运 Event 到 Channel 时的性能。
#### 2）Channel
type 选择 memory 时 Channel 的性能最好，但是如果 Flume 进程意外挂掉可能会丢失
数据。type 选择 file 时 Channel 的容错性更好，但是性能上会比 memory channel 差。
使用 file Channel 时 dataDirs 配置多个不同盘下的目录可以提高性能。
Capacity 参数决定 Channel 可容纳最大的 event 条数。transactionCapacity 参数决
定每次 Source 往 channel 里面写的最大 event 条数和每次 Sink 从 channel 里面读的最大
event 条数。transactionCapacity 需要大于 Source 和 Sink 的 batchSize 参数。
#### 3）Sink
增加 Sink 的个数可以增加 Sink 消费 event 的能力。Sink 也不是越多越好够用就行，
过多的 Sink 会占用系统资源，造成系统资源不必要的浪费。
batchSize 参数决定 Sink 一次批量从 Channel 读取的 event 条数，适当调大这个参数
可以提高 Sink 从 Channel 搬出 event 的性能。

## Flume 的事务机制
Flume 的事务机制（类似数据库的事务机制）：Flume 使用两个独立的事务分别负责从
Soucrce 到 Channel，以及从 Channel 到 Sink 的事件传递。
比如 spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的
事件全部传递到 Channel 且提交成功，那么 Soucrce 就将该文件标记为完成。
同理，事务以类似的方式处理从 Channel 到 Sink 的传递过程，如果因为某种原因使得
事件无法记录，那么事务将会回滚。且所有的事件都会保持到 Channel 中，等待重新传递。
## Flume 采集数据会丢失吗?
根据 Flume 的架构原理，Flume 是不可能丢失数据的，其内部有完善的事务机制，
Source 到 Channel 是事务性的，Channel 到 Sink 是事务性的，因此这两个环节不会出现数
据的丢失，唯一可能丢失数据的情况是 Channel 采用 memoryChannel，agent 宕机导致数据
丢失，或者 Channel 存储数据已满，导致 Source 不再写入，未写入的数据丢失。
Flume 不会丢失数据，但是有可能造成数据的重复，例如数据已经成功由 Sink 发出，
但是没有接收到响应，Sink 会再次发送数据，此时可能会导致数据的重复。