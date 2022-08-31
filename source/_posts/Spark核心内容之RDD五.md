---
title: Spark核心内容之RDD五
date: 2022-08-31 14:05:14
tags: [大数据,Spark]
---
# Spark核心内容RDD

Spark 计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于
处理不同的应用场景。三大数据结构分别是：
➢ RDD : 弹性分布式数据集
➢ 累加器：分布式共享只写变量
➢ 广播变量：分布式共享只读变量

<!--more-->

## 1.RDD
RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是 Spark 中最基本的数据
处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行
计算的集合。
➢ 弹性
⚫ 存储的弹性：内存与磁盘的自动切换；
⚫ 容错的弹性：数据丢失可以自动恢复；
⚫ 计算的弹性：计算出错重试机制；
⚫ 分片的弹性：可根据需要重新分片。
➢ 分布式：数据存储在大数据集群不同节点上
➢ 数据集：RDD 封装了计算逻辑，并不保存数据
➢ 数据抽象：RDD 是一个抽象类，需要子类具体实现
➢ 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的 RDD，在
新的 RDD 里面封装计算逻辑
➢ 可分区、并行计算
#### 1.1核心属性
![1](/img/2022-08-23/9.png)
![1](/img/2022-08-23/10.png)
![1](/img/2022-08-23/11.png)
#### 1.2执行原理
从计算的角度来讲，数据处理过程中需要计算资源（内存 & CPU）和计算模型（逻辑）。
执行时，需要将计算资源和计算模型进行协调和整合。
Spark 框架在执行时，先申请资源，然后将应用程序的数据处理逻辑分解成一个一个的
计算任务。然后将任务发到已经分配资源的计算节点上, 按照指定的计算模型进行数据计
算。最后得到计算结果。
![1](/img/2022-08-23/12.png)
![1](/img/2022-08-23/13.png)


# 1.3 RDD的使用

##### 1.3.1 RDD 创建
在 Spark 中创建 RDD 的创建方式可以分为四种：

从集合中创建 RDD，Spark 主要提供了两个方法：parallelize 和 makeRDD

从底层代码实现来讲，makeRDD 方法其实就是 parallelize 方法

###### 1.3.1.1 从集合（内存）中创建 RDD
```
val sparkConf =
 new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val rdd1 = sparkContext.parallelize(
 List(1,2,3,4)
)
val rdd2 = sparkContext.makeRDD(
 List(1,2,3,4)
)
rdd1.collect().foreach(println)
rdd2.collect().foreach(println)
sparkContext.stop()
```
###### 1.3.1.2 从外部存储（文件）创建 RDD
由外部存储系统的数据集创建 RDD 包括：本地的文件系统，所有 Hadoop 支持的数据集，比如 HDFS、HBase 等。
```
val sparkConf =
 new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
//按行读取文件
val fileRDD: RDD[String] = sparkContext.textFile("input")
fileRDD.collect().foreach(println)
sparkContext.stop()


//读取文件并且获取文件路径
val fileRDD: RDD[String]=sparkContext.wholeTextFiles("data")

```
###### 1.3.1.3 从其他 RDD 创建
主要是通过一个 RDD 运算完后，再产生新的 RDD。
###### 1.3.1.4 直接创建 RDD（new）
使用 new 的方式直接构造 RDD，一般由 Spark 框架自身使用


## RDD分区
默认情况下，Spark 可以将一个作业切分多个任务后，发送给 Executor 节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建 RDD时指定。记住，这里的并行执行的任务数量，并不是指的切分任务的数量，不要混淆了。
```
val sparkConf =
 new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val dataRDD: RDD[Int] =
 sparkContext.makeRDD(
 List(1,2,3,4),
 4)
val fileRDD: RDD[String] =
 sparkContext.textFile(
 "input",
 2)
fileRDD.collect().foreach(println)
sparkContext.stop()
⚫ 读取内存数据时，数据
```


