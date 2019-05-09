---
title: zookeeper的安装及其基本使用
date: 2019-05-05 22:52:52
tags: zookeeper
---

#分布式的应用服务协调框架

Google Chubby开源实现->基于 

zkcli.sh 客户端

## 使用zookeeper做服务的组件

* kafka
* hadoop  (master选举)
* Hbase   (**配置中心,集群监控** )
* dubbo  (注册中心 命名服务)
*  分布式锁  (比如 分布式id )
* 分布式协调通知

<!--more-->

## 创建节点

## 永久节点

`cerate /节点 数据`

### 临时节点

`cerate -e /节点 数据`

属于某个会话  

ßzookeeper退出后就消失了  

临时节点不能有子节点  

(常用)

### 顺序节点

`cerate -s /节点 数据`

有自己的序列

### tips

如果不加参数 默认持久在硬盘上 持久节点

![](/img/2019-5-5/1.png)

## 查看节点

`get  /节点`

`ls 可带节点` 

## 删除节点

`rmr /节点`

## 更新节点

`set /节点 222`

# 在java中调用zookeeper客户端api

* zookeeper原生客户端 (官方提供)
* zkClient (开源的zk客户端 在原生api上封装 文档少)
* Curator (netflix公司开源的客户端 apache顶级项目 更多时候用这个) **推荐**

### Curator创建

`retryPolicy 重试策略`

` ExponetialBackoffRetry(1000,3) .build`

![](/img/2019-5-5/2.png)



#   三种变更通知方式

watcher

* TreeCache 
* NodeCache (数据监听)
* PathAndChildCache (叶子节点 和本节点 监听)

### PathAndChildCache 监听

![](/img/2019-5-5/3.png)

![](/img/2019-5-5/4.png)



