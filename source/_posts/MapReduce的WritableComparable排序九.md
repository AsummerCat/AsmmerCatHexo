---
title: MapReduce的WritableComparable排序九
date: 2022-08-01 16:22:44
tags: [大数据,hadoop,MapReduce]
---
# MapReduce的WritableComparable排序
`MapTask`和`ReduceTask`均会对数据按照<font color='red'>Key</font>进行排序.
该操作属于`Hadoop`的默认行为.
<font color='red'>任何应用程序中的数据均会被排序,而不管逻辑上是不需要</font>

ps:
因为不进行排序的话,后面的`ReduceTask`无法对数据进行合并处理然后输出(需要每个key进行判断是否为同一个处理,效率低下)

默认排序是按照<font color='red'>字典顺序排序 (a,b,c,d....)</font>,且实现该排序的方法是<font color='red'>快速排序</font>

# 排序
## 部分排序
<!--more-->
```
MapReduce根据输入记录的键对数据集排序
保证输出的每个文件内部排序

比如:
自定义分区后,生成多个HDFS文件
每个HDFS文件中的数据 都按照倒序或者正序进行对数据的排序
```

## 全排序
最终输出结果只有一个文件,且文件内部有序;
```
实现方式:只设置一个ReduceTask.
```
但该方法在处理大型文件时效率极低;
因为一台机子处理所有文件,完全丧失了`MapReduce`所提供的并行架构;
