---
title: SparkStreaming自定义数据源三
date: 2022-08-31 14:31:35
tags: [大数据,Spark,SparkStreaming]
---
# SparkStreaming自定义数据源
`SparkStreaming`内置数据源并不多
可以自定义采集 mysql数据库数据等等

# 例子
需要继承 `Receiver`，并实现 onStart、onStop 方法来自定义数据源采集。
<!--more-->

## 使用自定义采集器
```
package day8

import org.apache.spark.SparkConf
import org.apache.spark.storage.StorageLevel
import org.apache.spark.streaming.receiver.Receiver
import org.apache.spark.streaming.{Seconds, StreamingContext}

import java.io.{BufferedReader, InputStreamReader}
import java.net.Socket
import java.nio.ByteBuffer
import java.nio.charset.StandardCharsets

/**
 * SparkStreaming自定义采集器
 */
object SparkStreamingFileStream {
  def main(args: Array[String]): Unit = {
    //1.初始化 Spark 配置信息
    val sparkConf = new SparkConf().setMaster("local[*]")
      .setAppName("StreamWordCount")
    //2.初始化 SparkStreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(5))
    //3.创建自定义 receiver 的 Streaming
    val lineStream = ssc.receiverStream(new CustomerReceiver("127.0.0.1", 9999))
    //4.将每一行数据做切分，形成一个个单词
    val wordStream = lineStream.flatMap(_.split("\t"))
    //5.将单词映射成元组（word,1）
    val wordAndOneStream = wordStream.map((_, 1))
    //6.将相同的单词次数做统计
    val wordAndCountStream = wordAndOneStream.reduceByKey(_ + _)
    //7.打印
    wordAndCountStream.print()
    //8.启动 SparkStreamingContext
    ssc.start()
    ssc.awaitTermination()
  }

  /**
   * SparkStreaming自定义采集器
   * 需要继承 Receiver，并实现 onStart、onStop 方法来自定义数据源采集。
   */
  class CustomerReceiver(host: String, port: Int) extends Receiver[String](StorageLevel.MEMORY_ONLY) {
    //采集器开关标识
    private var flag = true

    //最初启动的时候，调用该方法，作用为：读数据并将数据发送给 Spark
    override def onStart(): Unit = {
      new Thread(new Runnable {
        override def run(): Unit = {
          receive()
        }
      }, "Socket Receiver").start()
    }

    override def onStop(): Unit = {
      //自定义clone逻辑 比如mysql
      flag = false
    }

    //读数据并将数据发送给 Spark
    def receive(): Unit = {
      //创建一个 Socket
      var socket = new Socket(host, port)
      //定义一个变量，用来接收端口传过来的数据
      var input: String = null
      //创建一个 BufferedReader 用于读取端口传来的数据
      val reader = new BufferedReader(new InputStreamReader(socket.getInputStream, StandardCharsets.UTF_8))
      //读取数据
      input = reader.readLine()
      //当 receiver 没有关闭并且输入数据不为空，则循环发送数据给 Spark
      while (!isStopped() && input != null && flag) {
        //固定方法:推送 是Receiver里的实现
        store(input)
        input = reader.readLine()
      }
      //跳出循环则关闭资源
      reader.close()
      socket.close()
      //重启任务
      restart("restart")
    }
  }

}
```
