---
title: ElasticSearch笔记运维集群针对脑裂问题专门定制的重要参数(三十八)
date: 2020-09-03 22:17:35
tags: [ElasticSearch笔记]
---

# 最少master候选节点以及脑裂问题
重要的就这个参数
```
discovery.zen.minimum_master_nodes
```

# 脑裂原因
如果因为网络的故障,导致一个集群被划分成连两片,每片都有多个node,以及一个master.那么集群中就出现了两个master了,
但是因为master是集群中非常重要的一个角色,住在了集群状态的维护,以及shard的分配,因此如果有两个master的话,可能会导致破坏数据.

<!--more-->

# 为何要设置minimum_master_nodes参数
设置`minimum_master_nodes`就是告诉es直到有足够的master候选节点时,才可以选举出一个master,否则就不要选举出一个master.  
这个参数必须被设置为集群中master候选节点的quorum数量,
也就是大多数.至于quorum算法,就是:<font color="red">master候选节点数量/2+1</font>

  需要注意的是:生产环境最少需要3个节点   
  如果说3个节点钟两个节点故障的话,集群还是会宕机,但是这样能避免脑裂的问题  

  在es集群中是可以动态增加和下线节点的,所以可能随时会改变quorum.所以这个参数是可以通过api随时修改的,特别是在节点上线和下线的时候,都需要作出对应的修改.
  而且一旦修改过后,这个配置就会次就会保存下来.