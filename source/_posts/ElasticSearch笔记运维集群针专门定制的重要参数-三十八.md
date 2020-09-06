---
title: ElasticSearch笔记运维集群针专门定制的重要参数(三十八)
date: 2020-09-03 22:17:35
tags: [ElasticSearch笔记]
---

# 针对脑裂问题的重要参数

重要的就这个参数 用来处理出现脑裂的情况: 参数含义最小参与master选举的node数量

```
discovery.zen.minimum_master_nodes
```

## 脑裂原因

如果因为网络的故障,导致一个集群被划分成连两片,每片都有多个node,以及一个master.那么集群中就出现了两个master了,
但是因为master是集群中非常重要的一个角色,住在了集群状态的维护,以及shard的分配,因此如果有两个master的话,可能会导致破坏数据.

<!--more-->

## 为何要设置minimum_master_nodes参数

设置`minimum_master_nodes`就是告诉es直到有足够的master候选节点时,才可以选举出一个master,否则就不要选举出一个master.  
这个参数必须被设置为集群中master候选节点的quorum数量,
也就是大多数.至于quorum算法,就是:<font color="red">master候选节点数量/2+1</font>

  需要注意的是:生产环境最少需要3个节点   
  如果说3个节点钟两个节点故障的话,集群还是会宕机,但是这样能避免脑裂的问题  

  在es集群中是可以动态增加和下线节点的,所以可能随时会改变quorum.所以这个参数是可以通过api随时修改的,特别是在节点上线和下线的时候,都需要作出对应的修改.
  而且一旦修改过后,这个配置就会次就会保存下来.

api操作:

```
PUT /_cluster/settings
{
    "persistent":{
        "discovery.zen.minimum_master_nodes":2
    }
}

```

# 针对集群重启时的shard恢复耗时过长问题的重要参数

## 产生原因
集群重启的时候,可能由于某些原因部分shard没启动成功,剩下的节点会组成一个集群.之前的部分数据会重新复制.  
然后剩下的几个节点加入集群中后,发现本来是他们持有的shard被重新复制兵器放在其他node回档了,此时他们就会删除自己本地的数据.  
然后几区你又会开始进行shard的rebalance操作,将最早启动的node上的shard均匀分布到后来启动的node上去.  

这个过程,这些shard重新复制,移动,删除,再次移动的过程会,会大量的消耗网络和磁盘资源.  
对于数据量庞大的集群来说,可能导致每次集群重启时,都有TB级别的数据无端移动,可能导致集群启动会耗费很长时间.<font color="red">但是如果所有节点都可以等待集群中的所有节点都完全上线之后,所有的数据都有了以后,再决定是否要复制和移动shard,情况就会好很多</font>

## 自定义参数配置
```
gateway.recover_after_nodes: 8
这个参数可以让es直到有足够的node都上线之后,在开始shard recovery的过程

gateway.expected_nodes: 10
设置一个集群里至少要有多少个node

gateway.recover_after_time: 5m
等待那些node启动的时间

```
经过以上配置之后,es集群->   
等待至少8个node在线,然后等待最多5分钟.  
或者  
10个节点都在线,开始shrad recovery的过程.  
这样就可以避免少数node启动时,就立即开始shard recovery