---
title: Flink的窗口七
date: 2022-09-19 16:59:05
tags: [大数据,Flink]
---
# Flink的窗口

## 基础语法
```
stream.keyBy(<key selector>)
 .window(<window assigner>)
 .aggregate(<window function>)
```

<!--more-->
## 窗口分配器（Window Assigners）

### 时间窗口
```
stream.keyBy(...)
.window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
.aggregate(...)
```

### 计数窗口
底层 通过 全局窗口(Global Window)实现
```
  //2.计数窗口
    //我们定义了一个长度为 10 的滚动计数窗口，当窗口中元素数量达到 10 的时候，就会触发 计算执行并关闭窗口。
    stream.keyBy(_.user)
      .countWindow(10)

    //2.计数窗口
    //我们定义了一个长度为 10、滑动步长为 3 的滑动计数窗口。每个窗口统计 10 个数据，每隔 3 个数据就统计输出一次结果
    stream.keyBy(_.user)
      .countWindow(10, 3)
```

### 滚动窗口
```
    //1.滚动窗口
    stream.keyBy(_.user)
      //设置一个基于时间时间的滚动窗口 长度5秒的滚动窗口
      .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    //设置一个基于处理时间的滚动窗口 东八区的 00-24小时
    // .window(TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8))
```

### 滑动处理时间窗口
```
//1.滑动处理时间窗口
    //第一个参数:表示滑动窗口的大小，
    //第二个参数:表示滑动窗口的滑动步长。
    stream.keyBy(_.user)
      .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
```


## 窗口函数
窗口处理器处理好数据后->调用窗口函数进行数据的聚合转换

#### 增量聚合函数
来一个数据计算一次,最后统一输出结果

## 归约函数(ReduceFunction)
```
    stream.map(data => (data.user, 1))
      .keyBy(_._1)
      //设置一个基于时间时间的滚动窗口 长度5秒的滚动窗口
      .window(TumblingEventTimeWindows.of(Time.seconds(5)))
      //1.归约函数(ReduceFunction)
      //计数聚合
      .reduce((state, data) => (data._1, state._2 + data._2))
      .print()

```

##  AggregateFunction
AggregateFunction 可以看作是 ReduceFunction 的通用版本，这里有三种类型：输入类型
（IN）、累加器类型（ACC）和输出类型（OUT）。输入类型 IN 就是输入流中元素的数据类型；
累加器类型 ACC 则是我们进行聚合的中间状态类型；而输出类型当然就是最终计算结果的类
型了。
AggregateFunction 接口中有四个方法：
⚫ createAccumulator()：创建一个累加器，这就是为聚合创建了一个初始状态，每个聚
合任务只会调用一次。
⚫ add()：将输入的元素添加到累加器中。这就是基于聚合状态，对新来的数据进行进
一步聚合的过程。方法传入两个参数：当前新到的数据 value，和当前的累加器
accumulator；返回一个新的累加器值，也就是对聚合状态进行更新。每条数据到来之
后都会调用这个方法。
⚫ getResult()：从累加器中提取聚合的输出结果。也就是说，我们可以定义多个状态，
然后再基于这些聚合的状态计算出一个结果进行输出。比如之前我们提到的计算平均
值，就可以把 sum 和 count 作为状态放入累加器，而在调用这个方法时相除得到最终
结果。这个方法只在窗口要输出结果时调用。
⚫ merge()：合并两个累加器，并将合并后的状态作为一个累加器返回。这个方法只在
需要合并窗口的场景下才会被调用；最常见的合并窗口（Merging Window）的场景
就是会话窗口（Session Windows）。
```
package day6

import day2.Event
import org.apache.flink.api.common.eventtime.{SerializableTimestampAssigner, WatermarkStrategy}
import org.apache.flink.api.common.functions.AggregateFunction
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.assigners.SlidingProcessingTimeWindows
import org.apache.flink.streaming.api.windowing.time.Time

/**
 *
 * 窗口函数
 */
object WindowFunctionTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)

    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      ).assignTimestampsAndWatermarks(
      WatermarkStrategy.forMonotonousTimestamps[Event]()
        .withTimestampAssigner(
          new SerializableTimestampAssigner[Event] {
            override def extractTimestamp(element: Event, recordTimestamp: Long):
            Long = element.timestamp
          }
        )
    )

    //    stream.map(data => (data.user, 1))
    //      .keyBy(_._1)
    //      //设置一个基于时间时间的滚动窗口 长度5秒的滚动窗口
    //      .window(TumblingEventTimeWindows.of(Time.seconds(5)))
    //      //1.归约函数(ReduceFunction)
    //      //计数聚合
    //      .reduce((state, data) => (data._1, state._2 + data._2))
    //      .print()

    stream.keyBy(data => true)
      .window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))
      .aggregate(new AvgPv)
      .print()


  }
}

//自定义窗口函数
class AvgPv extends AggregateFunction[Event, (Set[String], Double), Double] {
  // 创建空累加器，类型是元组，元组的第一个元素类型为 Set 数据结构，用来对用户名进行去重
  // 第二个元素用来累加 pv 操作，也就是每来一条数据就加一
  override def createAccumulator(): (Set[String], Double) = (Set[String](), 0L)

  // 累加规则
  override def add(value: Event, accumulator: (Set[String], Double)):
  (Set[String], Double) = (accumulator._1 + value.user, accumulator._2 + 1L)

  // 获取窗口关闭时向下游发送的结果
  override def getResult(accumulator: (Set[String], Double)): Double =
    accumulator._2 / accumulator._1.size

  // merge 方法只有在事件时间的会话窗口时，才需要实现，这里无需实现。
  override def merge(a: (Set[String], Double), b: (Set[String], Double)):
  (Set[String], Double) = ???
}


```


