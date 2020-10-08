---
title: kafka笔记之Partition分区的副本机制-五
date: 2020-10-09 05:42:26
tags: [kafka笔记]
---

topic是存储消息的逻辑概念

partition
1. 每个topic可以划分多个分区
2. 相同topic下的不同分区的消息是不同的
3. 默认50个分区


# kafka笔记之消息分发策略
根据partition(分区列表)进行分发消息  
具体:  
根据key来分发消息  
默认算法(hash取模路由key)进行选择分区  


注意:  
kafka中一个partition不允许有并发,  
所以如果消费者大于分区数量会有部分消费者不会消费消息  
如果消费者小于分区数量会有部分消费者会消费多个分区消息
<!--more-->

<font color="red">增减 broker,consumer,partition 会导致Rebalance (重新负载)</font>  

## 分区分配策略
1. Range(范围) ->默认的  (按照字母分配消费者 如果消费者小于分区则消费者可能会消费多个分区消息 ; 如果消费者大于分区则会有部分消费者消费不到分区消息)
2. RoundBpbin(轮询) ->按照hashCode排序 按照轮询来排序  

可以通过 `partition.assignment.strategy`来指定分区分配策略


## kafka笔记之消息的存储策略
topic是逻辑上的保存概念  
物理保存路径是在 `partition`上

# 副本
副本的概念的就是对分区进行备份 起到了冗余的作用
意味着 每个分片可能存在多个副本
## 查看副本
```
在`zk`中
`/get /brokers/topics/topic_name/partition//partition_num/state`
```
## 副本的协同机制

```
返回内容
{"controller_epoch":1,"leader":1,"version":1,"leader_epoch":0,"isr":[1,0,2]}

leader: 表示在分区副本中 哪个是leader节点

isr: 维护当前分区的所有副本集 (in sync replicas) 
follwoers副本集必须要喝leader副本的数据在阀值范围内保持一致
有可能会出现数据同步延迟的情况  
那么follow节点就会被剔除,直到follow节点数据跟leader节点一致了 才会加入isr集合中
```

副本中也分为leader和follow  
leader副本: `负责接收客户端的消息写入和消息读取请求`  
follow副本: `负责从leader副本去读取数据  (不接受任何客户端的请求)`

注意:   
HW : 数据会标记为HW 之后 才能被客户端进行消费
LEO: 最新的一条数据记数位
remote LEO : 远程的


## 副本数据同步过程
初始化:   
1.将消息写入对应分区的log文件中,同时更新leader副本的LEO  

2.尝试去更新leader的HW的值,比较自己本身的LEO值和remote LEO的值.取最小的值作为HW  

更新/同步  
3.follow副本 获取leader数据进行同步

4.同步后 再更新leader上的HW 标记数据为客户端可消费




# kafka笔记之消息消费原理
## 消息的写入性能
1. 按照消息顺序写入
2. 零拷贝
3. LogSegment分段保存


# 监控工具
1. kafka monitor
2. kafka offset monitor
3. kafka -manager