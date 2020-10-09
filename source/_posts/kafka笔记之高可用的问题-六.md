---
title: kafka笔记之高可用的问题(六)
date: 2020-10-09 09:05:43
tags: [kafka笔记]
---

# 特性
```
kafka    天生分布式    每个topic可以指定多个partition    每个mq一个partition 也就是说 一个topic的消息会被分配到多个mq里面  比如 A 2个消息 B 3个消息

  |HA机制  : 0.8版本以后 有一个副本机制->生成副本partiion被在其他机子(follower)上会生成副本  并进行选举
    写入lader同步到follower

```

<!--more-->

## 如何保证kafka不会消息丢失?

```
消费者:   1.关闭自动提交offset 并且保证幂等性处理

消息队列: 2. 问题: 数据进入mq 还没同步就挂掉 选举上来的mq没有该数据
        | 解决方案: 
        ||2.1设置参数: 给topic设置 replication.factor参数>1 :保证每个partition必须至少有2个副本
        ||2.2 kafaka服务端设置min.insync.replicas参数 :必须大于1这个是要求一个leader至少感知到1个副本还跟自己联系
        
生产者:        
        ||2.3 在生产者端设置acks=all :要求每条数据必须写入所有副本之后才认为成功
        ||2.4 在生产者端设置retries=一个很大的值(MAX) :要求一旦写入失败,就无限重试,卡在这里

```