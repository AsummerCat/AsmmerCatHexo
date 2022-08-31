---
title: Spark的RDD文件读取和保存十
date: 2022-08-31 14:07:45
tags: [大数据,Spark]
---
# Spark的RDD文件读取和保存

## spark文件可以保存和读取

注意: 需要有hadoop的环境,不然文件相关的操作会报错
<!--more-->

```
package day3

import org.apache.spark.{SparkConf, SparkContext}

/**
 * RDD保存文件和读取文件
 */
object Spark03_RDD_Save_File {
  def main(args: Array[String]): Unit = {

    //创建scala和Spark框架的链接
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount")
    val sc = new SparkContext(sparkConf)

    //执行业务逻辑
    //1.创建RDD
    val rdd1 = sc.makeRDD(List(("a", 1), ("b", 2), ("c", 3), ("d", 4), ("e", 5)))

    //2. 保存rdd的数据
    rdd1.saveAsTextFile("SparkWordCount\\src\\data\\res1")
    rdd1.saveAsObjectFile("SparkWordCount\\src\\data\\res2")
    rdd1.saveAsSequenceFile("SparkWordCount\\src\\data\\res3")

    println("***********************")

    //3. 读取rdd保存的文件的数据
    val rdd3 = sc.textFile("SparkWordCount\\src\\data\\res1")
    println(rdd3.collect().mkString(","))
    val rdd4 = sc.objectFile[(String,Int)]("SparkWordCount\\src\\data\\res1")
    println(rdd4.collect().mkString(","))
    val rdd5 = sc.sequenceFile[String,Int]("SparkWordCount\\src\\data\\res1")
    println(rdd5.collect().mkString(","))


    //关闭链接
    sc.stop()
  }
}
```