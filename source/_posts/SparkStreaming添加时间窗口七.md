---
title: SparkStreaming添加时间窗口七
date: 2022-08-31 14:33:49
tags: [大数据,Spark,SparkStreaming]
---
# SparkStreaming添加时间窗口
可以将多个采集周期的数据合并一起处理

## WindowOperations
Window Operations 可以设置窗口的大小和滑动窗口的间隔来动态的获取当前Steaming 的允许状态。所有基于窗口的操作都需要两个参数，分别窗口时长以及滑动步长。
➢ 窗口时长：计算内容的时间范围；
➢ 滑动步长：隔多久触发一次计算。

<font color='red'>注意：这两者都必须为采集周期大小的整数倍。</font>
<!--more-->

## 案例
```
package day8

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * SparkStreaming 使用Window来创建时间窗口 进行多个采集周期数据 统一处理
 */
object SparkStreamingWindow {
  def main(args: Array[String]): Unit = {
    //1.初始化 Spark 配置信息
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamWordCount")
    //2.初始化 SparkStreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(3))
    //3.设置检查点
    ssc.checkpoint("/historyCache")

    //3.获取采集周期数据
    //某些场合下,需要历史采集周期数据,进行数据汇总 需要设置检查点来保留历史数据
    val lineStreams = ssc.socketTextStream("127.0.0.1", 9999)

    val data = lineStreams.map((_, 1))

    //语法:采集周期证书倍
    //data.window(Seconds(12), Seconds(6))

    //设置时间滑动窗口 3 秒一个批次，窗口 12 秒，滑步 6 秒。
    val wordCounts = data.reduceByKeyAndWindow((a: Int, b: Int) => (a + b), Seconds(12), Seconds(6))

    //7.开启任务
    ssc.start()
    ssc.awaitTermination()

  }
}
```