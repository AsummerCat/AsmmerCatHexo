---
title: Flink的值状态八
date: 2022-09-19 17:00:09
tags: [大数据,Flink]
---
# Flink的值状态


# KeyedState
以下为数据结构
## 1.值状态(ValueState)

例如:
```
  var valueState: ValueState[Event]  = getRuntimeContext.getState(new ValueStateDescriptor[Event]("my-value", classOf[Event]))
```
<!--more-->

### 案例
```
package day8

import day2.Event
import org.apache.flink.api.common.functions.{RichFlatMapFunction, RuntimeContext}
import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala._
import org.apache.flink.util.Collector


/**
 * Flink的值状态 (valueState)
 */
object ValueStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )


    val result = stream
      .keyBy(_.url)
      .flatMap(new MyMap)
      .print()

    env.execute()
  }

  /**
   * 自定义 RichFlatMapFunction
   */

  class MyMap extends RichFlatMapFunction[Event, String] {
    //定义一个值状态
    var valueState: ValueState[Event] = _

    override def getRuntimeContext: RuntimeContext = super.getRuntimeContext

    //开启函数初始化配置
    override def open(parameters: Configuration): Unit = {
      valueState = getRuntimeContext.getState(new ValueStateDescriptor[Event]("my-value", classOf[Event]))
    }

    //具体自定义函数
    override def flatMap(in: Event, collector: Collector[String]): Unit = {
      //对状态进行操作
      println("值状态为" + valueState.value())
      //更新值状态的值
      valueState.update(in)
    }
  }
}


```


## 2.列表状态 (ListState)
update(list) :传入一个列表values,直接对状态进行覆盖
add(value) : 在状态列表中添加一个元素
addAll(valus): 在状态列表中添加多个元素
```
package day8

import day2.Event
import org.apache.flink.api.common.functions.{RichFlatMapFunction, RuntimeContext}
import org.apache.flink.api.common.state.{ListState, ListStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala._
import org.apache.flink.util.Collector


/**
 * Flink的列表状态 (ListState)
 */
object ListStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )


    val result = stream
      .keyBy(_.url)
      .flatMap(new MyMap)
      .print()

    env.execute()
  }

  /**
   * 自定义 RichFlatMapFunction
   */

  class MyMap extends RichFlatMapFunction[Event, String] {
    //定义一个列表状态
    var stream1ListState: ListState[Event] = _

    override def getRuntimeContext: RuntimeContext = super.getRuntimeContext

    //开启函数初始化配置
    override def open(parameters: Configuration): Unit = {
      stream1ListState = getRuntimeContext.getListState(new ListStateDescriptor[Event]("stream-list", classOf[Event]))
    }

    //具体自定义函数
    override def flatMap(in: Event, collector: Collector[String]): Unit = {
      //对状态进行操作 添加
      stream1ListState.add(in)
    }
  }
}

```

## 3.映射状态(MapState)
跟普通map方法类似
```
package day8

import day2.Event
import org.apache.flink.api.common.functions.{RichFlatMapFunction, RuntimeContext}
import org.apache.flink.api.common.state.{MapState, MapStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala._
import org.apache.flink.util.Collector


/**
 * Flink的映射状态 (MapState)
 */
object MapStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )


    val result = stream
      .keyBy(_.url)
      .flatMap(new MyMap)
      .print()

    env.execute()
  }

  /**
   * 自定义 RichFlatMapFunction
   */

  class MyMap extends RichFlatMapFunction[Event, String] {
    //定义一个映射状态
    var mapState: MapState[Long, Event] = _


    override def getRuntimeContext: RuntimeContext = super.getRuntimeContext

    //开启函数初始化配置
    override def open(parameters: Configuration): Unit = {
      mapState = getRuntimeContext.getMapState(new MapStateDescriptor[String, Event]("window-pv", classOf[String], classOf[Event]))
    }

    //具体自定义函数
    override def flatMap(in: Event, collector: Collector[String]): Unit = {
      //对状态进行操作
      mapState.put(in.timestamp, in)
    }
  }
}

```

## 4.聚合状态（AggregatingState）
```
略
```

# 懒加载
先定义,运行的时候初始化
```
//定义一个值状态
    lazy val valueState: ValueState[Event] = getRuntimeContext.getState(new ValueStateDescriptor[Event]("my-value", classOf[Event]))

```

```
package day8

import day2.Event
import org.apache.flink.api.common.functions.{RichFlatMapFunction, RuntimeContext}
import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.util.Collector

/**
 * 懒加载 值状态
 */
object LazyValueStateTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )


    val result = stream
      .keyBy(_.url)
      .flatMap(new MyMap)
      .print()

    env.execute()
  }

  /**
   * 自定义 RichFlatMapFunction
   */

  class MyMap extends RichFlatMapFunction[Event, String] {
    //定义一个值状态
    lazy val valueState: ValueState[Event] = getRuntimeContext.getState(new ValueStateDescriptor[Event]("my-value", classOf[Event]))

    override def getRuntimeContext: RuntimeContext = super.getRuntimeContext

    //开启函数初始化配置
    override def open(parameters: Configuration): Unit = Unit

    //具体自定义函数
    override def flatMap(in: Event, collector: Collector[String]): Unit = {
      //对状态进行操作
      println("值状态为" + valueState.value())
      //更新值状态的值
      valueState.update(in)
    }
  }
}

```

# TTL
配置状态的 TTL 时，需要创建一个 StateTtlConfig 配置对象，然后调用状态描述器的enableTimeToLive()方法启动 TTL 功能。
```
    //TTL设置
    //1.过期时间为当前时间+TTL时间
    val ttlConfig = StateTtlConfig.newBuilder(Time.seconds(10))
      //2.设置更新类型
      //OnCreateAndWrite 表示只有创建状态和更改状态（写操作）时更新失效时间
      //OnReadAndWrite 则表示无论读写操作都会更新失效时间，也就是只要对状态进行了访问
      //默认OnCreateAndWrite
      .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
      //3.设置状态的可见性。所谓的“状态可见性”
      //NeverReturnExpired  失效就返回空
      //ReturnExpiredIfNotCleanedUp 失效后未被及时clear,还会返回数据
      .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
      .build()

    //定义一个映射状态

    lazy val mapState: MapState[Long, Event] = getRuntimeContext.getMapState(new MapStateDescriptor[String, Event]("window-pv", classOf[String], classOf[Event]).enableTimeToLive(ttlConfig))

    //失效时间 这部分内容需要加入到 new MapStateDescriptor知乎
    //    mapState.enableTimeToLive(ttlConfig)
```
### 完整代码
```
package day8

import day2.Event
import org.apache.flink.api.common.functions.{RichFlatMapFunction, RuntimeContext}
import org.apache.flink.api.common.state.{MapState, MapStateDescriptor, StateTtlConfig}
import org.apache.flink.api.common.time.Time
import org.apache.flink.api.scala.createTypeInformation
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.util.Collector


/**
 * Flink针对状态生存时间的设置
 * TTL
 */
object TtlTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val stream = env
      .fromElements(
        Event("Mary", "./home", 1000L),
        Event("Bob", "./cart", 2000L)
      )


    val result = stream
      .keyBy(_.url)
      .flatMap(new MyMap)
      .print()

    env.execute()
  }

  /**
   * 自定义 RichFlatMapFunction
   */

  class MyMap extends RichFlatMapFunction[Event, String] {
    //TTL设置
    //1.过期时间为当前时间+TTL时间
    val ttlConfig = StateTtlConfig.newBuilder(Time.seconds(10))
      //2.设置更新类型
      //OnCreateAndWrite 表示只有创建状态和更改状态（写操作）时更新失效时间
      //OnReadAndWrite 则表示无论读写操作都会更新失效时间，也就是只要对状态进行了访问
      //默认OnCreateAndWrite
      .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
      //3.设置状态的可见性。所谓的“状态可见性”
      //NeverReturnExpired  失效就返回空
      //ReturnExpiredIfNotCleanedUp 失效后未被及时clear,还会返回数据
      .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
      .build()

    //定义一个映射状态

    lazy val mapState: MapState[Long, Event] = getRuntimeContext.getMapState(new MapStateDescriptor[String, Event]("window-pv", classOf[String], classOf[Event]).enableTimeToLive(ttlConfig))

    //失效时间 这部分内容需要加入到 new MapStateDescriptor知乎
    //    mapState.enableTimeToLive(ttlConfig)


    override def getRuntimeContext: RuntimeContext = super.getRuntimeContext

    //开启函数初始化配置
    override def open(parameters: Configuration): Unit = {

    }

    //具体自定义函数
    override def flatMap(in: Event, collector: Collector[String]): Unit = {
      //对状态进行操作
      mapState.put(in.timestamp, in)

    }
  }
}


```

# 状态后端 (state的存储路径)
默认内存

```

```