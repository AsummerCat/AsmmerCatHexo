---
title: ElasticSearch笔记的相关机制(四)
date: 2020-08-11 14:03:42
tags: [elasticSearch笔记]
---

# ElasticSearch笔记shard与replica机制

![shard](/img/2020-08-02/2.png)
<!--more-->

## 注意
shard数量只能在创建index的时候修改
```
PUT /index
{
    "settings":{
        "number_of_shards":3,  //primary shard数量
        "number_of_replicas": 1 //每个primary shard都有一个replica shard
    }
}

```
#### replica
replica只承载用来备份数据,和读请求

# 容错机制

```
1.master选举

2.replica容错

3.数据恢复

```

# 并发问题 处理
### 乐观锁
es使用的是乐观锁方案  
有个`version`版本的字段来控制

每次增加或者修改`document`  
`_version`都会加一
```
带上版本号
例如:
POST /test_index/_update/1?version=1
{
    "doc": {
   	"name": "小1王1"
    }
}
```
也可以自定义一个字段当做`version`来控制
```
原乐观锁后加上 &vesion_type=external

跟原来es的vesion的差别是:
  自定义version>es的version的时候才能完成修改

例子:
POST /test_index/_update/1?version=9&version_type=external
{
    "doc": {
   	"name": "小1王1"
    }
}

按照上面的例子:
 本身es的version是1,我们自定义的3 
 3>1 所以能完成修改


```