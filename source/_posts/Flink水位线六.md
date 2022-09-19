---
title: Flink水位线六
date: 2022-09-19 16:58:14
tags: [大数据,Flink]
---
# Flink水位线

现在我们可以知道，水位线就代表了当前的事件时间时钟，而且可以在数据的时间戳基础上加一些延迟来保证不丢数据，这一点对于乱序流的正确处理非常重要。
我们可以总结一下水位线的特性：
⚫ 水位线是插入到数据流中的一个标记，可以认为是一个特殊的数据
⚫ 水位线主要的内容是一个时间戳，用来表示当前事件时间的进展
⚫ 水位线是基于数据的时间戳生成的
⚫ 水位线的时间戳必须单调递增，以确保任务的事件时间时钟一直向前推进
⚫ 水位线可以通过设置延迟，来保证正确处理乱序数据
⚫ 一个水位线 Watermark(t)，表示在当前流中事件时间已经达到了时间戳 t, 这代表 t 之
前的所有数据都到齐了，之后流中不会出现时间戳 t’ ≤ t的数据
水位线是 Flink流处理中保证结果正确性的核心机制，它往往会跟窗口一起配合，完成对乱序数据的正确处理
<!--more-->

## 水位线生成策略
基础方法
`assignTimestampsAndWatermarks()`，它主要用来为流中的数据分配时间戳，并生成水位线来指示事件时间。

## 设置水位线的周期间隔
默认200毫秒
```
    //水位线周期间隔
    env.getConfig.setAutoWatermarkInterval(60 * 1000L)
```

## Flink内置水位线生成策略

### 有序流
```
    //1.有序流  水位线生成策略 根据字段中的timestamp来排序
    stream.assignTimestampsAndWatermarks(
      WatermarkStrategy.forMonotonousTimestamps[Event]()
        .withTimestampAssigner(
          new SerializableTimestampAssigner[Event] {
            override def extractTimestamp(element: Event, recordTimestamp: Long):
            Long = element.timestamp
          }
        )
    )
```
### 乱序流
```
    //2.乱序流 插入水位线的逻辑
    stream.assignTimestampsAndWatermarks(
      //针对乱序流插入水位线，延迟时间设置为 5s
      WatermarkStrategy
        .forBoundedOutOfOrderness[Event](Duration.ofSeconds(5))
        .withTimestampAssigner(
          new SerializableTimestampAssigner[Event] {
            // 指定数据中的哪一个字段是时间戳
            override def extractTimestamp(element: Event, recordTimestamp: Long):
            Long = element.timestamp
          }
        )
    )
```
### 完整代码
```
package day5

import day2.Event
import org.apache.flink.api.common.eventtime.{SerializableTimestampAssigner, WatermarkStrategy}
import org.apache.flink.streaming.api.scala._

import java.time.Duration


/**
 * Flink内置水位线策略
 * 1. 有序流
 * 2. 乱序流
 */
object WaterMarkInnerTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )



    //水位线周期间隔
    env.getConfig.setAutoWatermarkInterval(60 * 1000L)

    //1.有序流  水位线生成策略 根据字段中的timestamp来排序
    stream.assignTimestampsAndWatermarks(
      WatermarkStrategy.forMonotonousTimestamps[Event]()
        .withTimestampAssigner(
          new SerializableTimestampAssigner[Event] {
            override def extractTimestamp(element: Event, recordTimestamp: Long):
            Long = element.timestamp
          }
        )
    )



    //2.乱序流 插入水位线的逻辑
    stream.assignTimestampsAndWatermarks(
      //针对乱序流插入水位线，延迟时间设置为 5s
      WatermarkStrategy
        .forBoundedOutOfOrderness[Event](Duration.ofSeconds(5))
        .withTimestampAssigner(
          new SerializableTimestampAssigner[Event] {
            // 指定数据中的哪一个字段是时间戳
            override def extractTimestamp(element: Event, recordTimestamp: Long):
            Long = element.timestamp
          }
        )
    )

    //事实上,有序流的水位线生成器本质上和乱序流是一样的，相当于延迟设为0的乱序流水位线生成器，两者完全等同：
    //WatermarkStrategy.forMonotonousTimestamps()
    //WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(0))
  }
}
```


# 迟到数据处理

1. 设置水位线延迟时间
2. 允许窗口处理迟到数据
3. 将迟到数据输出到侧数据流
```
package day7

import day2.Event
import org.apache.flink.api.common.eventtime.{SerializableTimestampAssigner, WatermarkStrategy}
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows
import org.apache.flink.streaming.api.windowing.time.Time

import java.time.Duration


/**
 * Flink迟到数据处理
 */
object LastDataWaterTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )



    //水位线周期间隔
    env.getConfig.setAutoWatermarkInterval(60 * 1000L)

    //水位线设置
    stream.assignTimestampsAndWatermarks(
      WatermarkStrategy
        //方式一.迟到数据处理 最大延迟时间设置为 5 秒钟
        .forBoundedOutOfOrderness[Event](Duration.ofSeconds(5))
        .withTimestampAssigner(
          new SerializableTimestampAssigner[Event] {
            override def extractTimestamp(element: Event, recordTimestamp: Long):
            Long = element.timestamp
          }
        )
    )

    // 定义侧输出流标签
    val outputTag = OutputTag[Event]("late")

    val result = stream
      .keyBy(_.url)
      .window(TumblingEventTimeWindows.of(Time.seconds(10)))
      // 方式二：允许窗口处理迟到数据，设置 1 分钟的等待时间
      .allowedLateness(Time.minutes(1))
      // 方式三：将最后的迟到数据输出到侧输出流
      .sideOutputLateData(outputTag)


    env.execute()


  }
}

```