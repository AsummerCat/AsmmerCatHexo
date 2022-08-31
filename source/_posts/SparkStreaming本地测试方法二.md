---
title: SparkStreaming本地测试方法二
date: 2022-08-31 14:30:47
tags: [大数据,Spark,SparkStreaming]
---
# SparkStreaming本地测试方法

## 使用RDD队列模拟监听端口

测试过程中，可以通过使用`ssc.queueStream(queueOfRDDs)`来创建 DStream，每一个推送到这个队列中的 RDD，都会作为一个 DStream 处理。

<!--more-->
## 例子:
```
package day7

import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.{Seconds, StreamingContext}

import scala.collection.mutable

object RDDStream {
  def main(args: Array[String]) {
    //1.初始化 Spark 配置信息
    val conf = new SparkConf().setMaster("local[*]").setAppName("RDDStream")

    //2.初始化 SparkStreamingContext
    //第一个参数 环境配置
    //第二个参数 采集周期 4秒
    val ssc = new StreamingContext(conf, Seconds(4))

    //3.创建 RDD 队列
    val rddQueue = new mutable.Queue[RDD[Int]]()

    //4.创建 QueueInputDStream
    val inputStream = ssc.queueStream(rddQueue, oneAtATime = false)

    //5.处理队列中的 RDD 数据
    val mappedStream = inputStream.map((_, 1))
    val reducedStream = mappedStream.reduceByKey(_ + _)

    //6.打印结果
    reducedStream.print()

    //7.启动采集器
    ssc.start()

    //8.循环创建并向 RDD 队列中放入 RDD
    for (i <- 1 to 5) {
      rddQueue += ssc.sparkContext.makeRDD(1 to 300, 5)
      Thread.sleep(2000)
    }

    //9.等待采集器的关闭
    ssc.awaitTermination()
  }
}
```