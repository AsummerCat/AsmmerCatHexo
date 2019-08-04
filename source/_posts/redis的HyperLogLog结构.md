---
title: redis的HyperLogLog结构
date: 2019-08-03 11:49:50
tags: [redis]
---

# redis的HyperLogLog结构

类似zset可以用来统计稍有误差

```java
Redis 提供了 HyperLogLog 数据结构就是用来解决
这种统计问题的。HyperLogLog 提供不精确的去重计数方案，虽然不精确但是也不是非常不
精确，标准误差是 0.81%，这样的精确度已经可以满足上面的 UV 统计需求了。
具备去重功能;
```



## 基本语法

pfadd: 添加 

pfcount: 记数

pfmerge:合并

<!--more-->

```
HyperLogLog 提供了两个指令 pfadd 和 pfcount，根据字面意义很好理解，一个是增加
计数，一个是获取计数。pfadd 用法和 set 集合的 sadd 是一样的，来一个用户 ID，就将用
户 ID 塞进去就是。pfcount 和 scard 用法是一样的，直接获取计数值。
127.0.0.1:6379> pfadd codehole user1
(integer) 1
127.0.0.1:6379> pfcount codehole
(integer) 1
127.0.0.1:6379> pfadd codehole user2
(integer) 1
127.0.0.1:6379> pfcount codehole
(integer) 2
127.0.0.1:6379> pfadd codehole user3
(integer) 1
```

```
> pfmerge test test1   合并test 和test2 原内容长度相等
OK
> pfcount test
6
> pfcount test1
3
```



# 注意事项

```
注意事项
HyperLogLog 这个数据结构不是免费的，不是说使用这个数据结构要花钱，它需要占据
一定 12k 的存储空间，所以它不适合统计单个用户相关的数据。如果你的用户上亿，可以算
算，这个空间成本是非常惊人的。但是相比 set 存储方案，HyperLogLog 所使用的空间那真
是可以使用千斤对比四两来形容了。
```

