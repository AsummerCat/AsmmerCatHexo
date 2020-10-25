---
title: ElasticSearch笔记运维集群搭建(三十七)
date: 2020-09-03 22:17:32
tags: [elasticSearch笔记]
---

# es目录结构
```
'bin': 存放es的一些可执行脚本,比如用于启动进程的elasticsearch命令,
以及用来安装插件的elasticsearch-plugin插件

'conf':用于存放es的配置文件,比如elasticsearch.yml

'data':用于存放es的数据文件,就是每个索引的shard的数据文件

'logs': 用于存放es的日志文件

'plugins': 用于存放es的插件

'script': 用于存放一些脚本文件 比如 搜索模板

```

<!--more-->

# master node
赋值维护整个集群的状态信息,也就是一些集群元数据信息,同事在`node`加入集群或者从集群中卸载时,重新分配`shard`,   
包括每次集群状态如果有改变的话,那么`master`都会负责将集群状态同步给所有的`node`.

# 集群服务发现机制
`zen discovery集群发现机制`
在生产环境中的多台机器上部署es集群,就涉及到了es的`discovery机制`,也就是集群中各个节点互相发现然后组成一个集群的机制,同事`discovery机制`也负责es集群的master选举

注意:  
 集群通信方式不是`client`跟`master`通信,而是`client`跟任何一个`node`进行通信,那个`node`再进行转发到对应的`node`来进行执行

 `master node`负责接收所有`cluster state`相关的变化信息,然后讲真改变后的最新`cluster state`推动给集群中的所有`data node`
每个`cluster`都有一份所有集群的状态,但是除了`master`之外的`node` 就只负责数据的存储和读写的,写入索引,搜索数据等等


# 集群部署方案

## 配置
elasticsearch.yml

1.JVM参数默认 是2G的
可手动修改 

2.修改`cluster name`的名字,默认是`elasticsearch`,在生产环境下一定要修改这个值,否则可能导致未知的`node`无端加入集群,造成集群运行异常

3.一般情况下 给集群划分出专门的`master`节点和`node`节点.  
一般建议`master node` 3个就够了  
具体设置:   
`matser配置` :`node.master:true ,node.data:false`  
剩下的node都设置为:`data node : node.master: false,node.data:true`  
10个以内的集群都开为`true`,大集群在拆分

4.设置es节点对外的暴露地址`network.host:192.168.31.222`   
不然默认是只允许本机访问


## 关于集群发现的配置
```
discovery.zen.ping.unicast.hosts: 用逗号分割主机列表

hosts.resolve_timeout: hostname被DNS解析为IP地址的timeout等待时长

discovery.zen.pin_timeout: 60s
默认3s,确保集群启动master的稳定性

discovery.zen.join_timeout:确保node稳定加入集群,增加join等待时长,如果一次join不上,默认重新20次

discovery.zen.matser.election.igonre_non_moaster_pings:
设置为'true',那么会强制区分master候选节点,如果'node'的'node master'设置为'false',
还来发送ping请求参与'master'选举,
那么这些'node'会被忽略掉,因为他们没有资格参与.


discovery.zen.minimum_master_nodes:
'预防集群脑裂'
参数用于设置对于一个新选举的master,要求必须有多少个matser候选的node去连接那个新选举的master,
而且还用于设置一个集群中必须拥有的matser候选的node,
如果这些要求没有达到,master node会被停止,然后重新选举一个新的master.
注意:必须设置为我们的master后端ndoe的quorum数量,
一般避免只有两个master候选node,因为2的quorum还是2,
如果在那个情况下,任何一个master候选节点宕机,集群就无法正常运作了.

```
简单来说,如果要让多个节点发现对方并且组成一个集群,
那么就得有一个中间的公共节点,然后不同的节点就发送请求到这些公共节点上,
接着通过这些公共节点交换各自的信息,进而让所有node感知到其他node的存在, 
并且进行通信组成一个集群.
以为这我们的'unicast list'只要提供少数几个node,就可以让新的node连接上  
如果配置了专门的matser节点,那么我们就只要配置这三个就可以了


## 集群故障的探查  
es有两种集群故障探查机制,  
第一种:是通过master进行的,master会ping集群中所有的其他node,确保他们是否存活着的  
第二种:每个node都会去ping master node来确保master node是存活的,否则就会发起一个选举过程
```
有下面三个参数用来配置集群故障的探查过程:

ping interval :每隔多长时间会ping一次node,默认是1s

ping_timeout :每次ping的timeout等待时长是多长时间,默认是30s

ping_retries :如果一个node被ping多少次都失败了,name认为node故障,默认是3次

```

## 集群状态更新
master node是集群中唯一可以对集群状态进行更新的node.  master node每次会处理一个集群状态的更新事件,然后将更新后的状态发布到集群中所有的node上去.每个node都会接受publish message,ack这个message.  但是不会应用这个更新.  
如果master没有在`discovery.zen.commit.timeout`指定时间内(默认30s),从至少`discovery.zen.minimum_master_nodes`个节点获取ack响应,name这次cluster state change时间就会被reject,不会应用;

但是一定在指定时间内,指定数量的node都返回了ack消息,那么cluster state就会被commit,然后一个message会被发送给所有的node.所有的node接收到这个commit message之后,接着才会将之前接收到的集群状态应用到自己本地的状态副本中去.  
接着master会等待所有节点响应是否更新自己本地副本状态成功,在一个等待超时时长内,如果接收到了响应(足够多的commit响应),那么就会继续处理内存queue中保存的下一个更新状态.  
`discovery.zen.publish_timeout`默认是30s,这个超时等待时长是从plublish cluster state开始计算的.
```
大致流程是:
1.matser接收到node集群的消息
2.发送集群消息给所有slave节点
3.salve节点确认消息但是不更新本地副本状态,返回ack给master
4.matser节点接收到slave节点发送ack消息后,再发送一次commit消息给所有slave节点
5.salve节点接收到commit消息后,更新本地副本状态,返回响应给master节点
6.matser节点接收到响应后,(足够多的commit响应)处理完成后,接着处理内存queue中保存的下一个更新状态;


discovery.zen.commit.timeout:
默认30s
master更新集群状态发送给node节点的超时时长.

discovery.zen.publish_timeout: 
默认30s
push消息的超时时长

```

## 不因为master宕机阻塞集群操作
如果要让集群正常运转,那么必须有一个`master`,还有`discovery.zen.minimum_master_nodes`指定数量的master候选node,都在运行.`discovery.zen.no_master_block`可以控制当master宕机,什么样的操作应该被拒绝.下面有两个选项
```
discovery.zen.no_master_block:all
一旦matser宕机,那么所有的操作都会被拒绝

discovery.zen.no_master_block:write
这是默认选项,所有的写操作都会被拒绝,但是读操作是被允许的

```


## 以上步骤 (组成最基础的集群)
```
cluster.name:
node.name:
netwok.host:
discovery.zen.ping.unicast.hosts:
以上四步配置后,能组成最基础的集群

```

# 组成生产集群的需要设置的一些重要参数

## 集群名称和节点名称
```
1.需要手动设置一个集群名称'cluster.name': xxx
避免开发人员本地es无意间加入到集群中

2.每个node都要手动设置一个名称,
每次重启节点都会随机分配,导致node名称每次重启都会变化
可以配置'node.name':指定节点名称名称
```

## 文件路径
数据目录,日志目录和插件目录
```
默认情况下,es会将plugin,log,还有data file都放在es的安装目录中.
这里有一个问题,就是在升级的时候,目录可能被覆盖,
导致我们丢失之前安装好的plugin,已有的log,已有的数据,已经配置好的配置文件

```
所以一般建议在`生产环境`中,必须都重新设置一下,放在es安装目录之外.
```
'path.data':  用于设置数据文件的目录

'path.logs':  用于设置日志文件的目录

'path.plugins': 用于设置插件存放的目录


注意:'path.data'可以指定多个目录,用逗号分隔即可.
如果多个目录在不同的磁盘上,那么这就是一个最简单的raid0的方式

一般建议的目录地址是:
elasticsearch.yml中修改

'path.data':  /var/log/elasticsearch

'path.logs':  /var/data/elasticsearch

'path.plugins': /var/plugins/elasticsearch
```

## 配置文件目录
es有两个配置文件  
1.`elasticsearch.yml`用于配置es  
2.`log4j.properties`用于es日志打印  
这些文件都被放在config目录下,默认就是`ES_HOME/config`  
可以通过以下命令重新设置
```
 启动的时候 附带配置文件目录
 ./bin/elasticsearch -Epath.conf=/path/to/my/config/
```