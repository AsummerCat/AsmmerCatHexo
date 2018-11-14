---
title: redis的基本使用-有图
date: 2018-11-13 10:08:46
tags:  redis
---
# 参考
[关于redis，学会这8点就够了](https://blog.csdn.net/middleware2018/article/details/80355418)

# redis的应用场景有哪些

1、会话缓存（最常用）  
2、消息队列，比如支付  
3、活动排行榜或计数  
4、发布、订阅消息（消息通知）  
5、商品列表、评论列表等  

---

# redis数据类型

Redis一共支持五种数据类：string(字符串)、hash(哈希)、list(列表)、set(集合)和zset（sorted set 有序集合）。

<!--more-->

## String类型
它是redis最基本的数据类型，一个key对应一个value，需要注意是一个键值最大存储512MB。

![String类型](/img/2018-11-13/redisToString.png)

## hash(哈希)
redis hash是一个键值对的集合， 是一个string类型的field和value的映射表，适合用于存储对象
![hash类型](/img/2018-11-13/redisToHash.png)


## list(列表)
是redis简单的字符串列表，它按插入顺序排序
![list类型](/img/2018-11-13/redisToList.png)

## set(集合)
是string类型的无序集合，也不可重复
![set类型](/img/2018-11-13/redisToSet.png)

## zset（sorted set 有序集合）
是string类型的有序集合，也不可重复  
sorted set中的每个元素都需要指定一个分数，根据分数对元素进行升序排序，如果多个元素有相同的分数，则以字典序进行升序排序，sorted set 因此非常适合实现排名
![Zset类型](/img/2018-11-13/redisToZset.png)

## 其他基本操作

```
slect           #选择数据库(数据库编号0-15)
quit             #退出连接
info             #获得服务的信息与统计
monitor       #实时监控
config get   #获得服务配置
flushdb       #删除当前选择的数据库中的key
flushall       #删除所有数据库中的key
```

## redis持久化

```
redis持久有两种方式:Snapshotting(快照),Append-only file(AOF)

Snapshotting(快照)

1、将存储在内存的数据以快照的方式写入二进制文件中，如默认dump.rdb中
2、save 900 1 

#900秒内如果超过1个Key被修改，则启动快照保存
3、save 300 10 

#300秒内如果超过10个Key被修改，则启动快照保存
4、save 60 10000 

#60秒内如果超过10000个Key被修改，则启动快照保存
 

Append-only file(AOF)

1、使用AOF持久时，服务会将每个收到的写命令通过write函数追加到文件中（appendonly.aof）
2、AOF持久化存储方式参数说明
    appendonly yes  

           #开启AOF持久化存储方式 
    appendfsync always 

         #收到写命令后就立即写入磁盘，效率最差，效果最好
    appendfsync everysec

         #每秒写入磁盘一次，效率与效果居中
    appendfsync no 

         #完全依赖OS，效率最佳，效果没法保证

```