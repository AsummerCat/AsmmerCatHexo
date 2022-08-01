---
title: 面试相关-SpringCloud
date: 2020-02-04 00:30:48
tags: [面试相关]
---

# 微服务

### 理论CAP理论和BASE理论

```
CAP理论:
一致性（Consistency）
可用性（Availability）
分区容错性（Partition tolerance）
最多达到两种 :
     分布式系统要么满足CA,要么CP，要么AP。无法同时满足CAP
     如eureka就是 AP   zk就是CP

```

```
BASE理论:
     基本可用（Basically Available）
     软状态（Soft state）
     最终一致性（Eventually consistent）
```

<!--more-->

### 分布式事务

```
分布式事务大概分为：
2pc（两段式提交）
3pc（三段式提交）
TCC（Try、Confirm、Cancel）
最大努力通知
XA
本地消息表（ebay研发出的）
半消息/最终一致性（RocketMQ）
```



## SpringCloud的基础功能

```
SpringCloud的基础功能：

服务治理：Spring Cloud Eureka

客户端负载均衡：Spring Cloud Ribbon

服务容错保护：Spring Cloud Hystrix  

声明式服务调用：Spring  Cloud Feign

API网关服务：Spring Cloud Zuul

分布式配置中心：Spring Cloud Config

SpringCloud的高级功能(本文不讲)：

消息总线：Spring  Cloud Bus

消息驱动的微服务：Spring Cloud Stream

分布式服务跟踪：Spring  Cloud Sleuth
```

# eureka源码

```
底层维护一个 hashMap
维护了注册到eureka的服务
```

###  注册流程

```
## 注册流程
1.注册接口  ApplicationResource.addInstance
2. 注册方法 serviceRegistry.register(this.registration)
3.将实例信息写入注册表的ConcurrentHashMap中
4.清理缓存 invalidateCache
```

### 获取注册接口

```
## 获取注册接口
1.从缓存获取实例 :responseCache.get(cacheKey)
2.从只读缓存获取 readOnlyCacheMap.get(key)         
      |  定时器默认30秒执行CacheUpdateTask任务更新读写缓存的内容进入只读缓存 
3.只读缓存不存在 ->去 读写缓存获取 readWriteCacheMap.get(key)    
      |  默认180秒过期
4.还是没有就直接从内存注册表里面获取
```

### eureka必须优化的参数

```
1.eureka.server.responseCacheUpdateIntervalMs = 3000
  ->注册中心读写缓存刷新到只读缓存的时间间隔
2.eureka.client.registryFetchIntervalSeconds = 3000
  ->客户端拉取注册表的时间间隔
3.eureka.client.leaseRenewalIntervalInSeconds = 30
  ->客户端发送心跳的间隔
4.eureka.server.evictionIntervalTimerInMs = 60000
  ->注册中心的线程定时发送检查心跳请求的间隔
5.eureka.instance.leaseExpirationDurationInSeconds = 90
  -注册中心 判断多久没有发送心跳 剔除服务
```



 # 分布式session方案

```
1.tomcat 做session复制的方案

2.spring session+redis 做分布式session

3.JWT or oauth2
```

# 分布式锁方案

```
1.zk
2.redis
理由:
zk优与redis

redis 分布式锁 需要不断去参数获取锁,比较消耗性能

zk 分布式锁 获取不到锁只要注册监听器就可以了

zk分布式锁 语义清晰 客户端挂了节点也就消失了 
redis还需要考虑其他因素,遍历上锁超时时间等
```

# 分布式事务方案

```
1. XA方案   两段式提交    
   -> 一阶段:询问(各数据库判断是否能完成该任务)  二阶段:执行
   -> 询问A数据库能否执行返回ok 询问B数据库能否执行返回ok 才算能执行完成  否则出现失败回滚
   -> 有一个事务管理器的概念 
   -> 弊端:严重依赖于数据库层面  效率低 绝不适用于高并发的情况下
   -> 实现:spring+JTA

2.TCC补偿方案
  ->try  confirm cancel
  -> try阶段: 各个数据库预留资源 或者锁定资源
  -> confirm阶段: 在各个服务中执行实际的操作
  -> cancel阶段: 失败回滚 各个服务进行补偿机制 ,也就是说完成的任务 需要进行回滚
  -> 弊端:严重依赖于你代码 来操作 回滚和补偿 可能导致补偿代码量巨大
  -> 适用于: 除非你真的对一致性要求很高 比如银行的资金处理 则可以使用TCC补偿 自己实现补偿方案
  -> 最后每个服务耗时比较短 不然会拉长整体效率
  
3. 本地消息表
   -> A系统 开启本地事务操作数据后 插入一条数据给本地消息表 ,接着发送消息给mq
   -> B系统接收到mq的消息,先插入一条数据给本地消息表再去本地事务中操作数据,如果消息存在则事务回滚 保证不重复消费
   -> B系统执行成功后,更新本地消息表及其A系统消息表
   -> 如果B系统操作失败,A系统会定期扫描消息表并且发送这个消息给B系统 保证最终一致性
   ->弊端: 严重依赖数据库的消息表来控制管理事务,高并发场景下 不容易扩展
  
  
4.可靠消息最终一致性方案   (靠谱点) (国内公司比较多)
   -> 消息队列实现   基于本地消息表改造  直接使用mq
   -> A系统 先给mq发送一条准备消息(这边有事务包裹) 发送失败就取消执行
   -> 发送mq准备消息成功,A执行本地事务,执行成功后,给mq确认消息 如果失败就回滚mq消息
   -> 如果A确认了mq的消息后,b接受到mq的请求,执行本地事务
   -> 如果B发生异常 无法执行的话   (B系统一定要保证一个幂等性)
       ->解决方案:
       ->     1.自动重试,直至成功
       ->     2.本地回滚后想办法通知A也回滚  
       ->     3.发送警报,由人工进行回滚处理和补偿
   
   
 5.最大努力通知事务
   -> A系统执行完本地事务 发送消息给mq
   -> 这里会有一个专门消费mq最大努力通知的服务,这个服务会消费mq请求,记录下来(如DB 或者放入队列),接着调用B系统的接口
   -> 如果B系统消费成功就ok了,如果失败,那么最大努力通知服务就会定时尝试访问B接口,重复N次,实在不行就放弃了 
   ->适用场景: 比如写入日志 等  消息少部分丢失也无所谓的情况
```



# 真题

### 如何保证接口的幂等性?

```
处理方案: 接口幂等性处理
    强检查: 查询id 订单号+业务场景(做主键) 不存在继续操作 存在 return   (重要场景)
    弱检查: redis缓存id 判断id 是否存在    (比如判断发送通知短信)
```

### 为什么要拆分成分布式?

```
1. 业务耦合度拆分 上万行代码拆分给不同的服务
2.迭代升级 不需要更新整个系统    只需要更新部分服务
3.一个人报错 所有开发一起报错

用分布式框架 因为: 有各种负载均衡 重试机制  rpc远程调用 等等...
```

### 跟dubbo的对比?

```
其实流程是差不多的

dubbo的RPC的性能会比(cloud)http的性能更好,并发能力更强,经过深度优化的RPC框架是 会好一点

dubbo只是一个单纯的服务框架 需要其他第三方组件配合
spring cloud是提供了一套完整的微服务方案  开箱即用
```

### 注册中心的选择方案是什么?

```
1.eureka :  可用性(AP)
   ->eureka 保证了可用性 某个eureka宕机后,服务仍可以使用  (如果挂了 可能会导致拉取的节点数据不是最新的)
   ->注册服务的时效性:  同步可能存在延时(注册服务和心跳检测) 默认延时30秒
   ->容量:rureka不适合大规模的集群上千节点 ,因为会造成网络带宽被占用 
   
   
2. zk: 一致性 (CP)
   ->zk的选举机制和集群同步 保证了强一致性 保证客户端及时刷新服务节点的信息
   ->注册服务的时效性:  因为是监听zk节点 所以服务的上线和下线都是秒级通知
   ->容量:zk也不适合大规模的集群上千节点 ,因为会造成网络带宽被占用 
   
3.dubbo使用zk, spring cloud使用eureka

4.Apollo 携程的注册中心 支持其他配置
5.diamond 阿里出品的一款
```

### 注册中心如何抗住上万服务上下线,如何优化? (自研)

```
1.分片存储
 ->注册中心 拆分成多个机器 每个机器存放不同数据 ,例如redis的集群模式
 ->这样避免了注册中心每个机子承受的压力
 ->客户端拉取数据去多台机子拉取
```

