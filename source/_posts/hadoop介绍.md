---
title: hadoop介绍
date: 2022-08-01 14:36:01
tags: [大数据,hadoop]
---

# hadoop介绍
## hadoop2.x架构
```

ResourceManager
NodeManager


NameNode
SecondaryNameNode
DataNode
```

<!--more-->

###     HDFS分布式文件系统
```
NameNode: 集群当中的主节点,主要用于管理集群当中的各种数据  (管理信息,元数据等)

SecondaryNameNode:  主要能用于hadoop当中元数据信息的辅助管理   定时对元数据进行备份

DataNode: 集群中的从节点,主要用于存储集群当中的各种数据

```

### Yarn资源调度系统
```
ResourceManager : 接收用户的计算请求任务,并负责集群的资源分配

NodeManager : 具体任务计算节点
```

### MapReduce 计算任务拆分
```
MapReduce计算需要的数据和产生的结果需要HDFS来进行存储

MapReduce的运行需要Yarn集群来提供资源调度
```


# 2.X常见端口号:
```
NameNode 内部通信端口: 8020 /9000

NameNode HTTP UI查询端口:  50070

MapReduce 查看执行任务端口:  8088

历史服务器通信端口:   19888
```

# 3.X常见端口号:
```
NameNode 内部通信端口: 8020 /9000 /9820

NameNode HTTP UI查询端口:  9870

MapReduce 查看执行任务端口:  8088

历史服务器通信端口:   19888
```

