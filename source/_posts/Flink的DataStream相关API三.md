---
title: Flink的DataStream相关API三
date: 2022-09-19 16:55:25
tags: [大数据,Flink]
---
# Flink的DataStream相关API

## 执行环境

智能化区分执行环境,统一的一行代码来区分本地环境或者集群环境

```
// 创建流执行环境
 val env = StreamExecutionEnvironment.getExecutionEnvironment
```

#### 执行模式
```
 // 批处理模式
val env = ExecutionEnvironment.getExecutionEnvironment

 // 创建流执行环境
val env = StreamExecutionEnvironment.getExecutionEnvironment
```
<!--more-->

## 源算子(Source)
读取数据的,返回`DataStream`流

#### 内置数据源
```
package day2

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer

import java.util.Properties

/**
 * source 多方式读取数据源 ->读取数据
 */
object SourceBoundedTest {
  def main(args: Array[String]): Unit = {
    //1.创建一个流式执行环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    //设置并行任务的数量为1
    env.setParallelism(1)

    //2.从指定元素中读取数据
    val value: DataStream[Int] = env.fromElements(1, 2, 3, 4)
    value.print()

    //3.从指定列表读取数据
    val value1 = env.fromCollection(List(Event1("Mary", "/.home", 1000L), Event1("Bob", "/.cart", 2000L)))
    //3.1 我们也可以不构建集合，直接将元素列举出来，调用 fromElements 方法进行读取数据：
    val value2 = env.fromElements(Event1("Mary", "/.home", 1000L), Event1("Bob", "/.cart", 2000L))
    value1.print()

    //4.从文件读取数据 可以是目录,也可以是文件
    val stream = env.readTextFile("clicks.csv")

    //5.从 Socket 读取数据
    val stream1 = env.socketTextStream("localhost", 7777)

    //6.从 Kafka 读取数据 需要增加依赖 flink-connector-kafka_${scala.binary.version}
    //目前最新版本 只支持 0.10.0 版本以上的 Kafka
    val properties = new Properties();
    properties.setProperty("bootstrap.servers", "hadoop102:9092")
    properties.setProperty("group.id", "consumer-group")
    properties.setProperty("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
    properties.setProperty("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer")
    properties.setProperty("auto.offset.reset", "latest")
    //创建一个 FlinkKafkaConsumer 对象，传入必要参数，从 Kafka 中读取数据
    //第一个参数: 主题TOP
    //第二个参数: 反序列化方式  使用的 SimpleStringSchema，是一个内置的 DeserializationSchema
    //第三个参数: 第三个参数是一个 Properties 对象，设置了 Kafka 客户端的一些属性
    val kafkaStream = env.addSource(new FlinkKafkaConsumer[String]("clicks", new SimpleStringSchema(), properties))
    kafkaStream.print("kafka")


    env.execute("获取内置数据源测试用例");
  }

}

//新增样例类
case class Event1(user: String, url: String, timestamp: Long)
```

#### 自定义读取数据源Source类(源算子)
实现 `SourceFunction` 接口，接口中的泛型是自定义数据源中的类型
```
package day2

import org.apache.flink.streaming.api.functions.source.SourceFunction
import org.apache.flink.streaming.api.functions.source.SourceFunction.SourceContext
import org.apache.flink.streaming.api.scala._

import java.util.Calendar
import scala.util.Random

/**
 * 自定义Source数据源 ->读取数据
 */
object CustomSourceBoundedTest {
  def main(args: Array[String]): Unit = {
    //1.创建一个流式执行环境
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    //设置并行任务的数量为1
    env.setParallelism(1)

    //2.从自定义数据源中读取数据
    val stream: DataStream[Event] = env.addSource(new ClickSource)
    stream.print()

    env.execute("获取自定义数据源测试用例");
  }

}

//新增样例类
case class Event(user: String, url: String, timestamp: Long)

// 实现 SourceFunction 接口，接口中的泛型是自定义数据源中的类型
class ClickSource extends SourceFunction[Event] {
  // 标志位，用来控制循环的退出
  var running = true

  //重写 run 方法，使用上下文对象 sourceContext 调用 collect 方法
  override def run(ctx: SourceContext[Event]): Unit = {
    // 实例化一个随机数发生器
    val random = new Random
    // 供随机选择的用户名的数组
    val users = Array("Mary", "Bob", "Alice", "Cary")
    // 供随机选择的 url 的数组
    val urls = Array("./home", "./cart", "./fav", "./prod?id=1", "./prod?id=2")
    //通过 while 循环发送数据，running 默认为 true，所以会一直发送数据
    while (running) {
      // 调用 collect 方法向下游发送数据
      ctx.collect(
        Event(
          users(random.nextInt(users.length)), // 随机选择一个用户名
          urls(random.nextInt(urls.length)), // 随机选择一个 url
          Calendar.getInstance.getTimeInMillis // 当前时间戳
        )
      )
      // 隔 1 秒生成一个点击事件，方便观测
      Thread.sleep(1000)
    }
  }

  //通过将 running 置为 false 终止数据发送循环
  override def cancel(): Unit = running = false
}
```



# 转换算子
## 基本转换算子
#### Map映射
```
package day3

import day2.Event
import org.apache.flink.api.common.functions.MapFunction
import org.apache.flink.streaming.api.scala._

/**
 * 转换算子的基本用法
 * Map映射函数的使用
 */
object MapTransTest {
  def main(args: Array[String]): Unit = {

    //1.Map映射
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )
    //1.使用匿名函数的方式提取 user 字段
    stream.map(_.user).print()
    //2.使用调用外部类的方式提取 user 字段
    stream.map(new UserExtractor).print()
    env.execute()
  }

  // 自定义映射函数 显式的实现 MapFunction 接口
  class UserExtractor extends MapFunction[Event, String] {
    override def map(value: Event): String = value.user
  }
}

```

#### 过滤（filter）
```
package day3

import day2.Event
import org.apache.flink.api.common.functions.FilterFunction
import org.apache.flink.streaming.api.scala._

/**
 * 转换算子的基本用法
 * Filter过滤函数的使用
 */
object FilterTransTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )
    //过滤出用户名是 Mary 的数据
    stream.filter(_.user.equals("Mary")).print()
    stream.filter(new UserFilter).print()
    env.execute()
  }

  //自定义过滤函数
  class UserFilter extends FilterFunction[Event] {
    override def filter(value: Event): Boolean = value.user.equals("Mary")
  }
}

```

#### 扁平映射（flatMap）
```
package day3

import day2.Event
import org.apache.flink.api.common.functions.FlatMapFunction
import org.apache.flink.streaming.api.scala._
import org.apache.flink.util.Collector

/**
 * 转换算子的基本用法
 * FlatMap扁平化函数的使用
 */
object FlatMapTransTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env.fromElements(
      Event("Mary", "./home", 1000L),
      Event("Bob", "./cart", 2000L)
    )
    stream.flatMap(new MyFlatMap).print()
    env.execute()
  }

  //自定义FlatMap扁平化函数  FlatMapFunction
  class MyFlatMap extends FlatMapFunction[Event, String] {
    override def flatMap(value: Event, out: Collector[String]): Unit = {
      //如果是 Mary 的点击事件，则向下游发送 1 次，如果是 Bob 的点击事件，则向下游发送 2 次
      if (value.user.equals("Mary")) {
        out.collect(value.user)
      } else if (value.user.equals("Bob")) {
        out.collect(value.user)
        out.collect(value.user)
      }
    }
  }
}

```

## 基本转换算子
#### 聚合算子（Aggregation）

##### 按键分区（keyBy）
keyBy()是聚合前必须要用到的一个算子。keyBy()通过指定键（key），可以将一条流从逻
辑上划分成不同的分区（partitions）

<font color='red'>这里只是区分并行度的分区(solt)</font>
```
package day3

import day2.Event
import org.apache.flink.streaming.api.scala._

/**
 * 转换算子的基本用法
 * KeyBy函数的使用 ,根据key进行分组
 */
object KeyByTransTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(4)
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L),
        Event("Aline", "./aline", 2000L)
      ) //指定 Event 的 user 属性作为 key
    val keyedStream = stream.keyBy(_.user)
    keyedStream.print()
    env.execute()
  }
}

```

#### 基础函数 sum ,min,max 等
注意首先要做`keyBy`函数操作才能调用
```
package day3

import day2.Event
import org.apache.flink.streaming.api.scala._

/**
 * 转换算子的基本用法
 * 基础函数 sum ,min,max 等
 * 注意首先要做keyBy函数操作才能调用
 */
object BaseTransTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env.fromElements(("a", 1), ("a", 3), ("b", 3), ("b", 4))
    stream.keyBy(_._1).sum(1).print() //对元组的索引 1 位置数据求和
    stream.keyBy(_._1).sum("_2").print() //对元组的第 2 个位置数据求和
    stream.keyBy(_._1).max(1).print() //对元组的索引 1 位置求最大值
    stream.keyBy(_._1).max("_2").print() //对元组的第 2 个位置数据求最大值
    stream.keyBy(_._1).min(1).print() //对元组的索引 1 位置求最小值
    stream.keyBy(_._1).min("_2").print() //对元组的第 2 个位置数据求最小值
    stream.keyBy(_._1).maxBy(1).print() //对元组的索引 1 位置求最大值
    stream.keyBy(_._1).maxBy("_2").print() //对元组的第 2 个位置数据求最大值
    stream.keyBy(_._1).minBy(1).print() //对元组的索引 1 位置求最小值
    stream.keyBy(_._1).minBy("_2").print() //对元组的第 2 个位置数据求最小值
    env.execute()
  }

}

object TransAggregationCaseClass {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env.fromElements(
      Event("Mary", "a", 2000L),
      Event("Mary", "b", 9000L),
      Event("aline", "g", 3000L),
      Event("Bob", "c", 2000L),
      Event("Bob", "d", 4000L),
      Event("Bob", "h", 3000L),
      Event("Mary", "b", 11000L),
      Event("Mary", "b", 9000L)
    )
    //注意这里是有限流输入的 所以每次都会输出记录
    // 使用 user 作为分组的字段，并计算最大的时间戳  会更新maxBy中对应的数据
    stream.keyBy(_.user).maxBy("timestamp").print()
    env.execute()
  }
}
```

