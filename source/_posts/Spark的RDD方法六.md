---
title: Spark的RDD方法六
date: 2022-08-31 14:05:44
tags: [大数据,Spark]
---
# Spark的RDD方法


## 1.Map
将数据逐条做转换, 可以是类型的转换,也可以是值的转换
```
val dataRDD :RDD[Int] = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD1 :RDD[Int]= dataRDD.map(num => { num =num*2})

或者
 val rdd1 = sc.parallelize(List(1,2,3,4,5), 2).map(_ * 2)
 
```
<!--more-->
## 2.mapPartitions
Map的升级版
以分区为单位处理数据
差别:
```
Map 算子主要目的将数据源中的数据进行转换和改变。但是不会减少或增多数据。 
MapPartitions 算子需要传递一个迭代器，返回一个迭代器，没有要求的元素的个数保持不变，
所以可以增加或减少数据
```
```
val dataRDD1: RDD[Int] = dataRDD.mapPartitions(datas => {datas.filter(_==2)} )
 
或者
分区内的数据都乘2
val dataRDD1: RDD[Int] = dataRDD.mapPartitions(datas => {datas.map(_*2)})
```

## 3.mapPartitionsWithIndex
在处理时同时可以获取当前分区索引。
```
val dataRDD1 = dataRDD.mapPartitionsWithIndex(
//分区号,迭代器
 (index, datas) => {
 datas.map(index, _)
 } )
```

## 4.flatMap
将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射
```
val dataRDD = sparkContext.makeRDD(List(List(1,2),List(3,4)),1)
val dataRDD1 = dataRDD.flatMap(list => list)

或
将字符按照空格切分 并且输出
val dataRDD = sparkContext.makeRDD(List("hello word", "hello Words"),1)
val dataRDD1 = dataRDD.flatMap(
 s => {
     s.split(" ")
 }
)
```

## 5.glom 转换成数组
将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
val dataRDD1:RDD[Array[Int]] = dataRDD.glom()
```

## 6.groupBy
将数据根据指定的规则进行分组,分区默认不变，但是数据会被打乱重新组合
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
val dataRDD1 = dataRDD.groupBy(_%2)
```

## 7.filter
将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)
val dataRDD1 = dataRDD.filter(_%2 == 0)
```

## 8.sample
根据指定的规则从数据集中抽取数据
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1)

// 抽取数据不放回（伯努利算法）
// 伯努利算法：又叫 0、1 分布。例如扔硬币，要么正面，要么反面。
// 具体实现：根据种子和随机算法算出一个数和第二个参数设置几率比较，小于第二个参数要，大于不
要
// 第一个参数：抽取的数据是否放回，false：不放回
// 第二个参数：抽取的几率，范围在[0,1]之间,0：全不取；1：全取；
// 第三个参数：随机数种子
val dataRDD1 = dataRDD.sample(false, 0.5)
// 抽取数据放回（泊松算法）
// 第一个参数：抽取的数据是否放回，true：放回；false：不放回
// 第二个参数：重复数据的几率，范围大于等于 0.表示每一个元素被期望抽取到的次数
// 第三个参数：随机数种子
val dataRDD2 = dataRDD.sample(true, 2)
```

## 9.distinct
将数据集中重复的数据去重
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4,1,2),1)
val dataRDD1 = dataRDD.distinct()
```

## 10.coalesce 压缩合并分区 缩减分区
根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4,1,2),6)

//默认情况下,不会将原先的分区进行打散处理
//如需打乱->加入第二个参数 ture
val dataRDD1 = dataRDD.coalesce(2)
val dataRDD1 = dataRDD.coalesce(2,true)
```

## 11.repartition 扩大分区
该操作内部其实执行的是 coalesce 操作
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4,1,2),2)
val dataRDD1 = dataRDD.repartition(4)
```

## 12.sortBy
该操作用于排序数据
```
val dataRDD = sparkContext.makeRDD(List(1,2,3,4,1,2),2)
//sortBy 默认升序
val dataRDD1 = dataRDD.sortBy(num=>num)
//sortBy 第二个参数表示降序
val dataRDD1 = dataRDD.sortBy(num=>num, false)
```

## 13.intersection 合并多个RDD 求交集
对源 RDD 和参数 RDD 求交集后返回一个新的 RDD
```
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.intersection(dataRDD2)
```

## 14.union  合并RDD取并集
对源 RDD 和参数 RDD 求并集后返回一个新的 RDD
```
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.union(dataRDD2)
```

## 15.subtract 求差集
以一个 RDD 元素为主，去除两个 RDD 中重复元素，将其他元素保留下来。求差集
```
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.subtract(dataRDD2)
```

## 16.zip
将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD
中的元素，Value 为第 2 个 RDD 中的相同位置的元素
```
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))
val dataRDD = dataRDD1.zip(dataRDD2)
```

## 17.partitionBy  根据分区重新分区排列数据
将数据按照指定 Partitioner 重新进行分区。Spark 默认的分区器是 HashPartitioner
```
import org.apache.spark.HashPartitioner

val rdd: RDD[(Int, String)] = sc.makeRDD(Array((1,"aaa"),(2,"bbb"),(3,"ccc")),3)

val rdd2: RDD[(Int, String)] =rdd.partitionBy(new HashPartitioner(2))
```

## 18.reduceByKey 性能比groupByKey优 (分组加聚合)
可以将数据按照相同的 Key 对 Value 进行聚合,
如果key只有一个不参与计算
```
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val dataRDD2 = dataRDD1.reduceByKey(_+_)
val dataRDD3 = dataRDD1.reduceByKey(_+_, 2)
```

## 19.groupByKey (仅分组)
将数据源的数据根据 key 对 value 进行分组
```
val dataRDD1 =
 sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val dataRDD2 = dataRDD1.groupByKey()
val dataRDD3 = dataRDD1.groupByKey(2)
val dataRDD4 = dataRDD1.groupByKey(new HashPartitioner(2))
```

## 20.aggregateByKey
将数据根据不同的规则进行分区内计算和分区间计算
```
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val dataRDD2 = dataRDD1.aggregateByKey(0)(_+_,_+_)
```
```
// TODO : 取出每个分区内相同 key 的最大值然后分区间相加
// aggregateByKey 算子是函数柯里化，存在两个参数列表
// 1. 第一个参数列表中的参数表示初始值
// 2. 第二个参数列表中含有两个参数
// 2.1 第一个参数表示分区内的计算规则
// 2.2 第二个参数表示分区间的计算规则
val rdd =
 sc.makeRDD(List(
 ("a",1),("a",2),("c",3),
 ("b",4),("c",5),("c",6)
 ),2)

//第一个参数是初始化值 ,用于默认比较的值
val resultRDD =
 rdd.aggregateByKey(10)(
 (x, y) => math.max(x,y),
 (x, y) => x + y
 )
resultRDD.collect().foreach(println)
```

## 21.foldByKey
当分区内计算规则和分区间计算规则相同时，aggregateByKey 就可以简化为 foldByKey
```
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val dataRDD2 = dataRDD1.foldByKey(0)(_+_)
```

## 22.combineByKey
将参与计算的第一个参数进行结构转换作为初始值计算,其他逻辑跟aggregateByKey一致
```
val combineRdd: RDD[(String, (Int, Int))] = input.combineByKey(
 (_, 1),
 (acc: (Int, Int), v) => (acc._1 + v, acc._2+ 1),
 (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
)
```
`AggregateByKey`：相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规
则可以不相同
`CombineByKey`:当计算时，发现数据结构不满足要求时，可以让第一个数据转换结构。分区
内和分区间计算规则不相同

## 23.sortByKey
在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口(特质)，返回一个按照 key 进行排序
的
```
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(true)
val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(false)
```

## 24.join
在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的

ps: 如果两个数据源key有多个相同可能会出现笛卡尔积
```
val rdd: RDD[(Int, String)] = sc.makeRDD(Array((1, "a"), (2, "b"), (3, "c")))
val rdd1: RDD[(Int, Int)] = sc.makeRDD(Array((1, 4), (2, 5), (3, 6)))
rdd.join(rdd1).collect().foreach(println)
```

## 25.leftOuterJoin
类似于 SQL 语句的左外连接
```
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val dataRDD2 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
val rdd: RDD[(String, (Int, Option[Int]))] = dataRDD1.leftOuterJoin(dataRDD2)
```

## 26.cogroup
在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的 RDD
```
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("a",2),("c",3)))
val dataRDD2 = sparkContext.makeRDD(List(("a",1),("c",2),("c",3)))
val value: RDD[(String, (Iterable[Int], Iterable[Int]))] = dataRDD1.cogroup(dataRDD2)
```

# RDD 行动算子

## 1.reduce
聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据
```
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 聚合数据
val reduceResult: Int = rdd.reduce(_+_)
```

## 2.collect
在驱动程序中，以数组 Array 的形式返回数据集的所有元素
```
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 收集数据到 Driver
rdd.collect().foreach(println)
```

## 3.count
返回 RDD 中元素的个数
```
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val countResult: Long = rdd.count()
```

## 4.first
返回 RDD 中的第一个元素
```
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val firstResult: Int = rdd.first()
println(firstResult)
```

## 5.take
返回一个由 RDD 的前 n 个元素组成的数组
```
vval rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val takeResult: Array[Int] = rdd.take(2)
println(takeResult.mkString(","))
```

## 6.takeOrdered
返回该 RDD 排序后的前 n 个元素组成的数组
```
val rdd: RDD[Int] = sc.makeRDD(List(1,3,2,4))
// 返回 RDD 中元素的个数
val result: Array[Int] = rdd.takeOrdered(2)
```

## 7.aggregate
分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合
```
val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 8)
// 将该 RDD 所有元素相加得到结果
//val result: Int = rdd.aggregate(0)(_ + _, _ + _)
val result: Int = rdd.aggregate(10)(_ + _, _ + _)
```

## 8.fold
折叠操作，aggregate 的简化版操作
```
val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4))
val foldResult: Int = rdd.fold(0)(_+_)
```

## 9.countByKey
统计每种 key 的个数
```
val rdd: RDD[(Int, String)] = sc.makeRDD(List((1, "a"), (1, "a"), (1, "a"), (2, 
"b"), (3, "c"), (3, "c")))
// 统计每种 key 的个数
val result: collection.Map[Int, Long] = rdd.countByKey()
```

## 10.save 相关算子
将数据保存到不同格式的文件中
```
// 保存成 Text 文件
rdd.saveAsTextFile("output")

// 序列化成对象保存到文件
rdd.saveAsObjectFile("output1")

// 保存成 Sequencefile 文件
rdd.map((_,1)).saveAsSequenceFile("output2")
```

## 11.foreach
分布式遍历 RDD 中的每一个元素，调用指定函数
```
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 收集后打印
rdd.map(num=>num).collect().foreach(println)
println("****************")
// 分布式打印
rdd.foreach(println)
```