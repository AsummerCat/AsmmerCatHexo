---
title: ElasticSearch笔记document的写入底层实现(十六)
date: 2020-08-11 14:12:19
tags: [ElasticSearch笔记]
---

# ElasticSearch笔记document的写入底层实现(十六) 


# 深入剖析document写入原理(buffer,segment,commit)
<font color="red">重点:底层原理</font>  
<font color="red">写入document:</font>
<!--more-->
```
# 写入原理
 总流程: 根据hash写入 ->有个协调节点 ->路由到对应的机子接收数据写入的请求,primary写入数据后->同步数据给replica机子
   ->primary和replica 都写入成功后 协调节点返回给客户端
   
1. 数据先写入buffer,在buffer中数据是搜索不到的,同时将数据写入translog日志文件(预写日志)

2.buffer满了之后或者到一定的时间,将buffer数据refresh刷新到一个segment file中,这边 不是直接写入磁盘,而是先进入os cache中

  |这个过程就是refresh.
  |默认是一秒refresh一次,所以es是实时的,数据写入一秒后才能被看到
  |数据只要进入了os cache中,buffer就会被清空,数据在translog里面已经持久化磁盘去一份了
  |只要数据进入os cache,此时就可以让这个segment file的数据对外提供搜索了
  
3. 重复1,2 ,不断的清除buffer,保留translog文件 ,这样translog越来越大.到达一定的长度后,就会触发commit操作

4. 强行将os cache的数据刷入到磁盘文件中去

5. translog 也是写入os cache中去的   默认5秒写入磁盘   (机器挂了最多也就是丢失5秒的数据)

6. 将现有的translog清空,重启一个translog,默认30分钟自动执行一次commit,如果log过大也是会执行的

7.如果是删除操作,commit的时候会生成一个.del文件,里面将某个doc标识为删除状态,搜索的时候就根据.del就知道删除了

8.如果是更新操作,将原来的数据标记为删除,再新写入一条数据

9.buffer每次refersh一次,就会产生一个segment file, 定期进行合并操作,

10.每次合并的时候就会将表示为删除的doc 物理删除,然后写入磁盘 ,删除旧的segment file
```


<font color="red">设置refresh的时间:</font>
```
POST /my_index/_refresh   
手动refresh 将buffer数据refresh刷新到一个segment file中,这边 不是直接写入磁盘,而是先进入os cache中
```
```
//创建索引的时候设置自动refresh时间
//默认是一秒refresh一次
PUT /my_index
{
    "settings":{
        "refresh_interval":"30s"
    }
}
```
<font color="red">设置异步translog的时间:</font>

```java
PUT /new_index/_settings
{
    "index.translog.durability":"async", 
    //异步执行预写预写日志
    "index.translog.sync_interval": "5s"
    //同步间隔 5秒一次
}
```
<font color="red">手动flush 写入磁盘:</font>  
清空预写日志和刷入磁盘

```
POST /new_index/_flush
```
<font color="red">手动合并segment:</font>  
默认会在后台执行segment merge操作,  
在mege的时候,被标记为deleted的document也会被彻底物理删除

```
POST /my_index/_optimize?max_num_segments=1
```

## 宕机后,数据恢复如何处理
```
机器被重启,disk上的数据并没有丢失,此时就会将
translog文件中的变更记录进行回放,
重新执行之前的各种操作,
在buffer中执行,
再重新输一个一个的segment到os cache中,等待下一次commit发生即可
```