---
title: HBase基础概念一
date: 2022-08-22 11:30:08
tags: [大数据,HBase]
---
# HBase基础概念

## HBase定义
HBase是一种分布式,可扩展,支持海量数据存储的NoSQL数据库
 <!--more-->
## HBase数据模型
逻辑上类似 关系型数据库,数据存储在一张表中,有行有列,但是从HBase的底层物理存储结构(K-V)来看更像是一个 稀松的,分布式的,持久的多维排序Map

##  HBase 物理存储结构
![image](/img/2022-08-18/1.png)
## 数据模型概念

#### Name Space
命名空间 = 关系型数据库中的`database`概念,每个命名空间下有很多表.
`HBase`有两个自带的命名空间`hbase`和`default`
```
hbase中存放的是HBase内置的表
default表是用户默认使用的命名空间。
```
#### Table
类似于关系型数据库的表概念。不同的是，HBase 定义表时只需要声明列族即可，不需
要声明具体的列。

#### Row
HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成，数据是按照 RowKey
的字典顺序存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重
要

#### Column
HBase 中的每个列都由 Column Family(列族)和 Column Qualifier（列限定符）进行限
定，例如 info：name，info：age。建表时，只需指明列族，而列限定符无需预先定义。

#### Time Stamp
用于标识数据的不同版本（version），每条数据写入时，系统会自动为其加上该字段，
其值为写入 HBase 的时间。

#### Cell
由{rowkey, column Family：column Qualifier, timestamp} 唯一确定的单元。cell 中的数
据全部是字节码形式存贮。


## 基本架构
![image](/img/2022-08-18/2.png)
![image](/img/2022-08-18/3.png)
```
三个主要角色
Master
Region Server 
Zookeeper

```
1) Master
   实现类为 HMaster，负责监控集群中所有的 RegionServer 实例。主要作用如下：
   （1）管理元数据表格 hbase:meta，接收用户对表格创建修改删除的命令并执行
   （2）监控 region 是否需要进行负载均衡，故障转移和 region 的拆分。
   通过启动多个后台线程监控实现上述功能：
   ①LoadBalancer 负载均衡器
   周期性监控 region 分布在 regionServer 上面是否均衡，由参数 hbase.balancer.period 控
   制周期时间，默认 5 分钟。
   ②CatalogJanitor 元数据管理器
   定期检查和清理 hbase:meta 中的数据。meta 表内容在进阶中介绍。
   ③MasterProcWAL master 预写日志处理器
   把 master 需要执行的任务记录到预写日志 WAL 中，如果 master 宕机，让 backupMaster读取日志继续干。
2) Region Server
   Region Server 实现类为 HRegionServer，主要作用如下: （1）负责数据 cell 的处理，例如写入数据 put，查询数据 get 等 （2）拆分合并 region 的实际执行者，有 master 监控，有 regionServer 执行。

3) Zookeeper HBase 通过 Zookeeper 来做 master 的高可用、记录 RegionServer 的部署信息、并且存储
   有 meta 表的位置信息。 HBase 对于数据的读写操作时直接访问 Zookeeper 的，在 2.3 版本推出 Master Registry
   模式，客户端可以直接访问 master。使用此功能，会加大对 master 的压力，减轻对 Zookeeper
   的压力。
4) HDFS
   HDFS 为 Hbase 提供最终的底层数据存储服务，同时为 HBase 提供高容错的支持。

