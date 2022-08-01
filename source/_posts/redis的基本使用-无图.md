---
title: redis的基本使用-无图
date: 2018-11-13 10:08:46
tags:  [redis]
---
# 参考
[简单说一下redis](https://blog.csdn.net/bushanyantanzhe/article/details/79485441)

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

>String : string是redis最基本的类型，一个key对应一个value。string类型是二进制安全的。意思是redis的string可以包含任何数据。比如图片或者序列化的对象。String类型是Redis最基本的数据类型，一个键最大能存储512MB。  
常用命令:   
① get、获取存储在指定键中的值   
② set、设置存储在指定键中的值   
③ del、删除存储在指定键中的值（这个命令可以用于所有的类型）   
使用场景:利用incr生成id,decr减库存, 缓存–过期时间设置，模拟session



## hash(哈希)
redis hash是一个键值对的集合， 是一个string类型的field和value的映射表，适合用于存储对象

>hash : Redis hash 是一个键值(key=>value)对集合。Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。  
常用命令：  
①hset、在散列里面关联起指定的键值对  
②hget、获取指定散列键的值  
③hgetall、获取散列包含的所有键值对  
④hdel、如果给定键存在于散列里面，那么移除这个键 
使用场景:购物车 


## list(列表)
是redis简单的字符串列表，它按插入顺序排序

>list: Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。 
常用命令：  
① rpush、将给定值推入列表的右端  
② lrange、获取列表在指定范围上的所有值  
③ lindex、获取列表在指定范围上的单个元素  
③ lpop、将最左边的元素推出  
③ rpop、将最右边的元素推出   
③ trim key 1 0、移除list   
使用场景: 多任务调度队列

## set(集合)
是string类型的无序集合，也不可重复

>set: Redis的Set是string类型的无序集合。集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。  
常用命令：  
①sadd、将给定元素添加到集合  
②smembers、返回集合包含的所有元素  
③sismember、检查指定元素是否存在于集合中  
③srem 、移除元素  
使用场景: 微博关注数 

## zset（sorted set 有序集合）
是string类型的有序集合，也不可重复  
sorted set中的每个元素都需要指定一个分数，根据分数对元素进行升序排序，如果多个元素有相同的分数，则以字典序进行升序排序，sorted set 因此非常适合实现排名

>zset: Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。  
常用命令：  
①zadd、将一个带有给定分值的成员添加到有序集合里面  
②zrange、根据元素在有序排列中所处的位置，从有序集合里面获取多个元素  
②zcard、用来查看集合个数  
②ZRANGE page_rank 0 -1 WITHSCORES、遍历显示集合全部   
②zrem、用来移除元素 


使用场景:排行榜 



