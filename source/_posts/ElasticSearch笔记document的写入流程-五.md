---
title: ElasticSearch笔记document的写入流程(五)
date: 2020-08-11 14:05:04
tags: [ElasticSearch笔记]
---

# document数据路由原理
### document路由到shard上
```
一个index的数据会被分为多片,每片都在一个shard中,
说一说,一个document只能存在于一个shard中/

当客户端创建document的时候,es此时就需要决定说,
这个document是放在这个index的哪个shard上

这个过程就称为 document routing, 数据路由.

```
<!--more-->
### 路由算法
```
shard=hash(routing)%number_of_primary_shards

举例:
一个index有3个 primary shard  P0,P1,P2
每次增删改查一个document的时候,都会带过来一个routing number ,默认就是这个document的_id


当然也可以手动设置路由id

一个index上的数据会被封为多片,每片都在一个shard中

PUT /test_index/_doc/1?routing=age
{
  "name":"小明",
  "age":1
}


```

###  primary shard为啥不可变
```
因为跟路由算法有关

如果更改了primary shard数量 会导致路由不正确
间接导致数据丢失 因为路由后的shard找不到该document

```

---
---

# document增删改查内部实现流程
##### 流程逻辑:
```
1.客户端选中一个node发送请求过去,这个node就是coordinating node (协调节点)

2.协调节点对document进行路由,将请求转发给对应的node(有primary shard)

3.实际的node上的primary shard处理请求,然后将数据同步到replica node

4.协调节点,如果发现primary node和所有replica node都搞定之后,就返回相应结果给客户端
```
### document<增>内部实现流程 写一致性的剖析  
```
1. 一致性: one(primary shard) , all(all shard) , quorum(default)
我们在发送任何一个增删改操作的时候,
比如    put /index/type/id?consistency=one,
都可以带上一个consistency参数,指明我们想要的一致性是什么
 one: 要求这个写操作,只要一个primary shard是active活跃可用,就可以运行
 
 all: 要求这个写操作,必须所有primary shard和replica shard都是活跃的
 
 quorum: 要求这个写操作,必须是大部分的shard都是活跃的,可用的,才可以会信这个写操作 (默认的)
 
 
 
2.qurum机制,写之前必须确保大多数的shard都可用,
int((primary+number_of_replicas)/2)+1,
当number_of_replicas>1时才生效

举个例子: 3个primary shard,number_of_replicas=1, 3+3*1总共有6个shard
qurum=((3+1)/2)+1
所以,要求6个shard至少有3个shrad是可用的

3.如果节点数少于quorum数量,可能导致quorum不齐全,进而导致无法执行任何写操作

4.quorum不齐全时,wait,默认1分钟,timeout,100,30s
 会等待1分钟,期望quorum齐全,超时就写入失败
 可以添加参数
 put /index/type/id?timeout=30 来缩短等待时间

```

# document<查询>内部实现流程
```
对于读请求 
   协调节点不一定会把请求转发到primary shard上,
replica shard 也是可以处理读请求的

使用采取轮询算法
   协调节点接收到请求后,会根据轮询自动分配到shard上,尽可能得到负载均衡的效果
   
   
```
#### 读流程
```
1.客户端发送请求到任意一个node,成为协调节点

2.协调节点对document进行路由,将请求转发到对应的node,此时会使用轮询算法
在primary shard以及其所有replica中随机选择一个,
让读请求负载均匀

3.接收请求的node返回document给协调节点

4.协调节点返回document给客户端

5.特殊情况: 
   document如果还在建立索引过程中,可能只有primary shard有,
任何一个replica shard都没有,此时可能会导致无法读取到document,
但是document完成索引建立之后,primary shard和replica shard就都有了


```

# bulk api
```
1.bulk中的每个操作都可能要转发到不同的node的shard去执行

2.如果采用比较良好的josn数组格式可能会导致消耗更多的内存
所以采用了这种两行的形式

直接按行切割字符串

3.优势:不需要将json数组解析为一个jsonArray对象,形成一份大数据的拷贝,浪费内存空间


```