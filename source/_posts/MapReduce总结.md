---
title: MapReduce总结
date: 2022-08-01 16:25:03
tags: [大数据,hadoop,MapReduce]
---
# MapReduce总结

# 1.InputFormat 数据输入
```
默认实现类是: TextInputFormat
TextInputFormat实现逻辑: 一次读一行文本,然后将该行的起始偏移量作为key,行内容作为value返回;

CombineTextInputFormat: 可以把多个小文件合并成一个切片处理,提高处理效率;
```
# 2.逻辑处理接口MapTask:Mapper
```
用户根据业务需求实现其中三个方法
 
setup()  初始化
map()  用户的业务逻辑
clearnup()  关闭资源
``` 
<!--more-->

# 3.Shuffle机制
```
位于Map任务之后,Reduce任务之前 ;
用来进行分片,压缩,排序等处理
```
###  3.1 Partition分区
默认分区
```
默认分区 HashPartitioner,
默认按照key的hash值%numreduceTask
```

自定义分区
```
在 job中设置自定义Partitioner

job.setPartitionerClass(自定义分区.claas);

需要搭配 指定NumReduceTasks 
job.setNumReduceTasks(2);

 用来确认
 最终输出几个HDFS文件,根据什么规则进行将结果路由输出到不同文件中
 
```


###   3.2 WritableComparable排序
```
1.部分排序: 每个输出的文件内部有序

2.全排序: 一个reduce,对所有数据大排序
(1.2这两部分是默认的)

3.二次排序 
自定义排序,实现WritableComparable接口,
重写compareTo方法;
```

# 3. OutputFormat 数据输出
```
1.默认TextOutputFormat 按行输出到文件

2.可自动以实现输出OutputFormat
```
