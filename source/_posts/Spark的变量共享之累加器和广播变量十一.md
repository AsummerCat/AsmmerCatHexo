---
title: Spark的变量共享之累加器和广播变量十一
date: 2022-08-31 14:08:43
tags: [大数据,Spark]
---
# Spark的变量共享之累加器和广播变量


## Spark的累加器全局共享只写变量 (累加器)
累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，传回 Driver 端进行 merge。
#### 基础代码
```
package day4

import org.apache.spark.{SparkConf, SparkContext}

/**
 * 累加器的使用 基础  var sum = sc.longAccumulator("sum")
 */
object Spark_Acc_Base {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)

    //执行业务逻辑
    //1.创建RDD

    val rdd = sc.makeRDD(List(1, 2, 3, 4, 5))
    // 声明累加器 系统内置三中累加器 1.longAccumulator 2.doubleAccumulator   3.collectionAccumulator
    var sum = sc.longAccumulator("sum")
//    var sum = sc.doubleAccumulator("sum")
//    var sum = sc.collectionAccumulator("sum")

    //系统自带了double和collection的累加器
    rdd.foreach(
      num => {
        // 使用累加器
        sum.add(num)

      })
    // 获取累加器的值
    println("sum = " + sum.value)

  }

}

```
<!--more-->
#### 自定义累加器
```
package day4

import org.apache.spark.util.AccumulatorV2
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.mutable

/**
 * 自定义累加器的使用 AccumulatorV2
 */
object Spark_Acc_custom {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)

    //执行业务逻辑
    //1.创建RDD
    val rdd = sc.makeRDD(List("hello", "world", "test"))

    //2.创建自定义累加器对象
    val acc = new Spark_CustomACC()
    //3.注册并且构建累加器
    sc register(acc, "custom")

    //4.处理业务逻辑
    rdd.foreach(
      num => {
        // 使用累加器
        acc.add(num)

      })
    // 获取累加器的值
    println("sum = " + acc.value)

  }


  // 自定义累加器
  // 1. 继承 AccumulatorV2，并设定泛型
  // 2. 重写累加器的抽象方法
  class Spark_CustomACC extends AccumulatorV2[String, mutable.Map[String, Long]] {
    var map: mutable.Map[String, Long] = mutable.Map()

    // 累加器是否为初始状态
    override def isZero: Boolean = {
      map.isEmpty
    }

    // 复制累加器
    override def copy(): AccumulatorV2[String, mutable.Map[String, Long]] = {
      new Spark_CustomACC
    }

    // 重置累加器
    override def reset(): Unit = {
      map.clear()
    }

    // 向累加器中增加数据 (In)
    override def add(word: String): Unit = {
      // 查询 map 中是否存在相同的单词
      // 如果有相同的单词，那么单词的数量加 1
      // 如果没有相同的单词，那么在 map 中增加这个单词
      var wordCount = map.getOrElse(word, 0L) + 1L
      map.update(word, wordCount)
    }

    // 合并累加器
    override def merge(other: AccumulatorV2[String, mutable.Map[String, Long]]):
    Unit = {
      val map1 = map
      val map2 = other.value
      // 两个 Map 的合并
      map = map1.foldLeft(map2)(
        (innerMap, kv) => {
          innerMap(kv._1) = innerMap.getOrElse(kv._1, 0L) + kv._2
          innerMap
        }
      )
    }

    // 返回累加器的结果 （Out）
    override def value: mutable.Map[String, Long] = map
  }

}

```




## Spark的累加器全局共享只读变量 (广播变量)
广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个
或多个 Spark 操作使用。
#### 基础代码
```
package day4

import org.apache.spark.{SparkConf, SparkContext}

/**
 * 广播变量的使用(只读变量) 基础 sc.broadcast(list)
 */
object Spark_Bc_Base {
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)

    //执行业务逻辑
    //1.创建RDD

    val rdd = sc.makeRDD(List(1, 2, 3, 4, 5))

    //2.新增广播变量
    val list = List(2, 4)
    val bc = sc.broadcast(list)

    rdd.map(
      num => {
        // 获取广播变量的内容
        //3.判断rdd中的值 如果为2,4则*10倍
        if (bc.value.contains(num)) {
          num * 10
        } else {
          num
        }
      }).collect().foreach(println)

  }

}

```