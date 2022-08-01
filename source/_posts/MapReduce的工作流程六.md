---
title: MapReduce的工作流程六
date: 2022-08-01 16:21:00
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的工作流程

1.输入待处理的文本

2.切片处理 ->在客户端submit()之前,对任务分配进行一个规划

3.提交任务->对集群进行提交
```
生成
1.job.split 
job的切片信息

2.wc.jar  
运行的jar包 
(本地模式无jar包,如果是集群name就是发送给Yarn RM运行,那么就需要提交jar包)

3.job.xml
里面存放该job运行的相关参数
```

<!--more-->

4.计算出MapTask数量
```
开启一个MrAppmaster (Yarn的集群) 读取切片信息

根据提供切片信息,开启对应的MapTask;
```

5.MapTask任务运行
```
InputFormat进行读取输入的文件,

//业务逻辑处理
读取完成后输出给Mapper,
然后输出给环形缓存区(默认100M),分区和排序(归并排序)
```

6.ReduceTask运行
`ReduceTask`主动去拉取对应分区中的`MapTask`的数据(shuffle机制会进行分区),然后将多个`MapTask`的数据,再进行一次归并排序合并在一起处理,这样能保证相同的`key`进去`Reduce`方法
```
当所有的MapTask完成后,启动ReduceTask
ps: 
  1.如果数据量小,可MapTask全部处理完成后再运行ReduceTask
  2.如果数据量大(很多MapTask)可合并部分数据优先启动ReduceTask

//启动数量的ReduceTask,并告知ReduceTask处理数据范围(数据分区)

然后输入给Reduce进行处理;
```

7.根据Reducer的OutPutFormat
```
由
  OutPutFormat
     ->RecordWriter
          ->Write(k,v)
          最终进行输出数据文件,落地到HDFS上

例如: Part-r-0000000
```
