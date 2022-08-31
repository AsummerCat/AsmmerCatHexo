---
title: Spark自定义分区规则九
date: 2022-08-31 14:07:23
tags: [大数据,Spark]
---
# Spark自定义分区规则

自定义分区需要继承`Partitioner`
实现内部的两个方法

<!--more-->

## 例子:
```
package day3

import org.apache.spark.{Partitioner, SparkConf, SparkContext}

/**
 * RDD 设置检查点
 */
object Spark03_RDD_My_Partitioner {
  def main(args: Array[String]): Unit = {

    //创建scala和Spark框架的链接
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)


    //执行业务逻辑
    //2.创建RDD 并行RDD(业务列表,调用CPU核数分片处理)
    val rdd1 = sc.parallelize(List(("nba", "xxxx"), ("cba", "xxxx"), ("wna", "xxxx"), ("pna", "xxxx"), ("gna", "xxxx")), 2)
    
    //3.自定义分区
    val rdd2 = rdd1.partitionBy(new My_Partitioner)

    rdd2.collect().foreach(print)

    //关闭链接
    sc.stop()
  }
}

/**
 * 自定义分区器
 * 1.继承Partitioner
 * 2.重写方法
 */
class My_Partitioner extends Partitioner {
  //分区数量
  override def numPartitions: Int = 3

  //返回数据的分区索引,从0开始 ,根据数据的key来判断分区
  override def getPartition(key: Any): Int = {
    if (key == "nba") {
      0
    } else if (key == "nba") {
      1
    } else {
      2
    }
  }
}
```