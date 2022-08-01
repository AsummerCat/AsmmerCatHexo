---
title: hadoop调优手册之MapReduce常用调优参数
date: 2022-08-01 16:32:28
tags: [大数据,hadoop,MapReduce,hadoop调优手册]
---
# hadoop调优手册之MapReduce常用调优参数


## MapReduce跑的慢的原因
MapReduce程序效率的瓶颈在于两点:
(1).计算机性能
CPU ,内存,磁盘,网络

(2).I/O操作优化
1.数据倾斜
2.Map运行时间太长,导致Reduce等待过久
3.小文件过多

<!--more-->

# 常用调优参数
配置存放位置: `mapred-site.xml`

#### MapTask阶段优化

1) 自定义分区,减少数据倾斜
```
定义类,继承Partitioner接口,重写getPartition方法
```
2)减少溢写的次数
```
mapreduce.task.io.sort.mb
Shuffle的环形缓存区,默认100m,可以提高到200m

mapreduce.map.sort.spill.percent
环形缓存区溢出的阀值,默认80%,可以提高到90%
```
3)增加每次Merge合并次数
```
mapreduce.task.io.sort.factor 
默认为10,可以提高到20
```
4)在不影响业务结果的前提条件下可以提前采用Cornbiner
```
提前在shuffle中溢出部分数据操作

job.setCombinerClass(xxxReducer.class)
```

5)为了减少磁盘IO,可以采用Snappy或者LZO压缩
```
//开启map端输出压缩
conf.setBoolean("mapreduce.map.output.compress","true");
//设置map端输出压缩方式  参数:压缩方式,编码方式
conf.setClass("mapreduce.map.output.compress.codec",SnappyCodec.claas,CompressionCodec.class);

//设置map端输出压缩方式  参数:压缩方式,编码方式

```

6) 默认MapTask内存上限为1024MB
```
mapreduce.map.memory.mb 
可以根据128m数据对应1G内存原则提高该内存  (等于栈内存)
```

7) 控制MapTask堆内存大小
```
mapreduce.map.java.opts  
(如果内存不够,报错,OOM)  可以设置跟6的一致
```

8) 默认MapTask的CPU核数1
```
mapreduce.map.cpu.vcores
计算密集型任务可以增加CPU核数
```

9) 异常重试
```
mapreduce.map.maxattempts
每个MapTask最大重试次数,一旦重试次数超过该值,则认为Maptask运行失败;
默认值为4. 根据机器性能适当提高
```

#### ReduceTask阶段优化

1) 每个Reduce去Map中拉取数据的并行数
```
默认值是5; 可以提高到10

mapreduce.reduce.shuffle.parallelcopies
```

2) Buffer大小占Reduce可用内存的比例
```
默认值 0.7 ,可以提高到0.8

mapreduce.reduce.shuffle.input.buffer.percent
```

3) Buffer中数据达到多少比例开始写入磁盘
```
默认值0.66,可以提高到0.75

mapreduce.reduce.shuffle.merge.percent

```

4) 默认ReduceTask内存上限1024MB (可以当做栈内存)
```
根据128m数据对应1G内存原则,适当提高内存到4-6g

mapreduce.recduce.memory.mb
```

5) 控制ReduceTask堆内存大小 (可以设置为跟4的一样)
```
mapredue.reduce.java.opts

如果内存不够,报错OOM;
```

6) 默认ReduceTask的CPU核数1个
```
mapreduce.reduce.cpu.vcores

可以提高到2-4个
```

7)每个ReduceTask最大重试次数
```
mapreduce.reduce.maxattempts

一旦重试次数超过该值,这认为reduceTask运行失败,默认值:4
```

8) 当MapTask完成的比例达到该值后才会为ReduceTask申请资源
```
默认值0.05
```

9)如果一个Task在一定时间内没有任何进入,无读取新数据,也没有输出数据,则认为Task处于Block状态
```
mapreduce.task.timeout (单位毫秒)

可以强制设置一个超时时间,默认是600000(10分钟),
如果你的程序对每条输入数据的处理时间过长,建议将该参数调大;

```

10) 如果可以Reduce,尽可能不用
