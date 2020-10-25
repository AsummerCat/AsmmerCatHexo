---
title: ElasticSearch笔记优化建议-四十五
date: 2020-09-10 21:52:39
tags: [elasticSearch笔记]
---

# document查询和结构的优化

## 搜索或者查询结果不要返回过大的结果集
   只是用来搜索,而不是用来存取大批量数据的  
   如果要做大批量结果的查询,记得考虑用scroll api


## 避免超大的document

`http.max_context_length`的默认值是100mb,意味着你写入document时不能超过100mb.否则es拒绝写入  
如果修改了该参数,`lucene`引擎还是有一个2gb的最大限制

超大的document还会耗费更多的网络资源,内存资源和磁盘资源

可以选择拆分数据 这样可以优化搜索的体验 分词查询会更少匹配到无关内容

<!--more-->

## 避免稀疏的数据

```
100个document 有20个字段

密集数据: 100个document 20个字段 都有值

稀疏数据: 100个document  有的document只有2个字段 有的document有50个field

```
避免方案:
1.避免将没有任何关联性的数据写入同一个索引
结构不同的数据最好写入不同的索引
2.对document的结构进行规范化/标准化
3. 避免使用多个types
4. 对稀疏的field禁用norms和doc_values


# 索引写入性能优化
## (1)用bulk批量写入
```
减少多次网络请求

可以用压测来测出es平稳的写入的性能
可以用bulk 100 200 300 400 来测试

```

## (2)使用多线程将数据写入es
单线程发送bulk请求是无法最大化es集群写入的吞吐量的  
如果要利用集群的所有资源,就需要多线程并发将数据bulk写入集群中.为了更好的利用集群的资源,这样多线程并发写入  
可以减少每次底层磁盘`fsync`的次数和开销.一样,可以对单个es节点的单个shard做压测,一旦发现es返回了`TOO MANY_REQUESTS`的错误,也就是`EsRejectedExecutionException`,那么就说明es已经达到一个并发写入的瓶颈了,此时我们就知道最多只能支撑这么搞的并发写入了

## (3)增加refresh间隔
默认的`refresh`间隔是1s,  
`index.refresh.interval`参数可以设置,这样会强迫es每秒中都将内存中的数据写入磁盘中,创建一个新的`segment file`.  
正是这个间隔,让我们每次写入数据后,1s以后才能看到.  
但是如果我们将这个间隔调大,比如30s,可以接受写入的数据30s后才看到,那么我们就可以获取更大的写入吞吐量,因为30s内都是写内存的,每隔30s才会创建一个`segment file`

## (4)禁止refresh和replica
如果我们要一次性加载大批量的数据进es,可以先禁止`refresh`和`replica`复制,  
将`index.refresh_interval`设置为-1,  
将`index.number_of_replicas`设置为0即可.  
这可能会导致我们的数据丢失,因为没有`refresh`和`replica`机制了.  
但是此时写入的数据会非常快,一旦写入完成之后,可以将`refresh`和`replica`修改回正常的状态



## (5)禁止swapping交换内存
 禁止之后,减少内存被磁盘吃掉

## (6)给filesystem cache更多的内存
给filesystem cache被用来执行更多的IO操作,如果我们能给filesystem cache更多的内存资源,那么es的写入性能会好很多

## (7)使用自动生成的id
如果手动设置一个id,那么es需要每次确认这个id是否存在,这个过程是比较耗时的.  
   如果我们使用自动生成的id,那么es就可以跳过这个步骤,写入性能会更好


## (8)用性能更好的硬件
我们可以给`filesystem cache`更多的内存,也可以用使用SSD替代机械硬盘,避免使用NAS等网络存储,考虑使用RAID矩阵来提高磁盘并行读写效率,等等

## (9) index buffer
如果我们要进行非常重的高并发写入操作,那么最好将`index buffer`调大一些,  
`indices.memory.index_buffer_size`这个可以调大一些,但是对于每个shard来说,最多给`512mb`,因为再大性能就没什么提升了.  
es会将这个设置作为每个`shard`共享的`index buffer`,那些特别活跃的`shard`会更多的使用这个bufferr.  
默认参数的值是`10%`,也就是`jvm heap`的`10%`,  
如果我们给`jvm heap`分配10g内存,那么这个`index buffer`就有1gb ,对于两个shard共享来说,是足够的