---
title: SparkStreaming转换DStream数据为RDD操作六
date: 2022-08-31 14:33:21
tags: [大数据,Spark,SparkStreaming]
---
# SparkStreaming转换DStream数据为RDD操作

Transform 允许 DStream 上执行任意的 RDD-to-RDD函数。即使这些函数并没有在 DStream的 API中暴露出来，通过该函数可以方便的扩展 SparkAPI。该函数每一批次调度一次。其实也就是对 DStream中的 RDD 应用转换。

<!--more-->

## Transform示例
```
package day8

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * SparkStreaming 将DStream数据转换为RDD来操作
 */
object SparkStreamingTransform {
  def main(args: Array[String]): Unit = {
    //1.初始化 Spark 配置信息
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamWordCount")

    //2.初始化 SparkStreamingContext
    //第一个参数 环境配置
    //第二个参数 采集周期 3秒
    val ssc = new StreamingContext(sparkConf, Seconds(3))

    //3.通过监控端口创建 DStream，读进来的数据为一行行
    val lineStreams = ssc.socketTextStream("127.0.0.1", 9999)

    //4.将DStream数据转换为RDD来操作
    val rdd = lineStreams.transform(rdd => rdd)

    //将每一行数据做切分，形成一个个单词
    val wordStreams = rdd.flatMap(_.split(" "))

    //将单词映射成元组（word,1）
    val wordAndOneStreams = wordStreams.map((_, 1))

    //将相同的单词次数做统计
    val wordAndCountStreams = wordAndOneStreams.reduceByKey(_ + _)

    //打印
    wordAndCountStreams.print()

    //启动采集器
    ssc.start()

    //等待采集器的关闭
    ssc.awaitTermination()
  }
}

```