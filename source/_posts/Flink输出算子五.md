---
title: Flink输出算子五
date: 2022-09-19 16:57:09
tags: [大数据,Flink]
---
# Flink输出算子
Flink 的 DataStream API 专门提供了向外部写入数据的方法：addSink。与 addSource 类似，addSink 方法对应着一个“Sink”算子，主要就是用来实现与外部系统连接、并将数据提交写入的；Flink 程序中所有对外的输出操作，一般都是利用 Sink 算子完成的。
<!--more-->
```
stream.addSink(new SinkFunction(…))
```

### 输出到文件
⚫ 行编码：StreamingFileSink.forRowFormat t(basePath,rowEncoder)。
⚫批量编码：StreamingFileSink.forBulkFormat(basePath,bulkWriterFactory)。
```
package day4

import day2.Event
import org.apache.flink.api.common.serialization.SimpleStringEncoder
import org.apache.flink.core.fs.Path
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy
import org.apache.flink.streaming.api.scala._

import java.util.concurrent.TimeUnit

/**
 * flink文件流输出 文件addSink
 */
object SinkToFileTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(4)
    val stream = env.fromElements(
      Event("Mary", "./home", 1000L),
      Event("Bob", "./cart", 2000L),
      Event("Alice", "./prod?id=100", 3000L),
      Event("Alice", "./prod?id=200", 3500L),
      Event("Bob", "./prod?id=2", 2500L),
      Event("Alice", "./prod?id=300", 3600L),
      Event("Bob", "./home", 3000L),
      Event("Bob", "./prod?id=1", 2300L),
      Event("Bob", "./prod?id=3", 3300L)
    )

    //文件路径 ,编码,字符集
    val fileSink = StreamingFileSink.forRowFormat(new Path("src/ioutput"), new SimpleStringEncoder[String]("UTF-8"))
      //通过.withRollingPolicy()方法指定“滚动策略”
      .withRollingPolicy(
        DefaultRollingPolicy.builder()
          //至少包含15分钟内的数据
          .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
          //最近5分钟没有收到新数据
          .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
          //当文件大小已达到1GB
          .withMaxPartSize(1024 * 1024 * 1024)
          .build()
      ).build

    stream.map(_.toString).addSink(fileSink)
    env.execute()
  }
}
```


### 输出到kafka
需要导入依赖
```
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
```
```
package day4

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer

import java.util.Properties

/**
 * 生成数据addSink推送到kafka
 */
object SinkToKafkaTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val properties = new Properties()
    properties.put("bootstrap.servers", "hadoop102:9092")
    val stream = env.readTextFile("input/clicks.csv")
    stream.addSink(new FlinkKafkaProducer[String]("clicks", new SimpleStringSchema(), properties))
    env.execute()
  }
}
```

### 输出到 Redis
需要导入依赖
```
        <dependency>
            <groupId>org.apache.bahir</groupId>
            <artifactId>flink-connector-redis_2.12</artifactId>
            <version>1.1.0</version>
        </dependency>
```
```
package day4

import day2.{ClickSource, Event}
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.redis.RedisSink
import org.apache.flink.streaming.connectors.redis.common.config.FlinkJedisPoolConfig
import org.apache.flink.streaming.connectors.redis.common.mapper.{RedisCommand, RedisCommandDescription, RedisMapper}

object SinkToRedisTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val conf = new FlinkJedisPoolConfig.Builder().setHost("hadoop102").build()
    env.addSource(new ClickSource)
      //参数一:Jedis 的连接配置。
      //参数二:Redis 映射类接口，说明怎样将数据转换成可以写入 Redis 的类型
      .addSink(new RedisSink[Event](conf, new MyRedisMapper()))
    env.execute()
  }
}

class MyRedisMapper extends RedisMapper[Event] {
  override def getKeyFromData(t: Event): String = t.user

  override def getValueFromData(t: Event): String = t.url

  override def getCommandDescription: RedisCommandDescription = new RedisCommandDescription(RedisCommand.HSET, "clicks")
}
```

### 输出到 Elasticsearch
需要导入依赖
```
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-elasticsearch6_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
```
```
package day4

import day2.Event
import org.apache.flink.api.common.functions.RuntimeContext
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.connectors.elasticsearch.{ElasticsearchSinkFunction, RequestIndexer}
import org.apache.flink.streaming.connectors.elasticsearch6.ElasticsearchSink
import org.apache.http.HttpHost
import org.elasticsearch.client.Requests

import java.util

/**
 * 生成数据addSink推送到es
 */
object SinkToEsTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env.fromElements(
      Event("Mary", "./home", 1000L),
      Event("Bob", "./cart", 2000L),
      Event("Alice", "./prod?id=100", 3000L),
      Event("Alice", "./prod?id=200", 3500L),
      Event("Bob", "./prod?id=2", 2500L),
      Event("Alice", "./prod?id=300", 3600L),
      Event("Bob", "./home", 3000L),
      Event("Bob", "./prod?id=1", 2300L),
      Event("Bob", "./prod?id=3", 3300L)
    )

    //httpHosts：连接到的 Elasticsearch 集群主机列表。
    //elasticsearchSinkFunction：这并不是我们所说的 SinkFunction，而是用来说明具体处理逻辑、准备数据向 Elasticsearch 发送请求的函数。
    val httpHosts = new util.ArrayList[HttpHost]()
    httpHosts.add(new HttpHost("hadoop102", 9200, "http"))
    val esBuilder = new ElasticsearchSink.Builder[Event](
      httpHosts,
      new ElasticsearchSinkFunction[Event] {
        override def process(t: Event, runtimeContext: RuntimeContext,
                             requestIndexer: RequestIndexer): Unit = {
          val data = new java.util.HashMap[String, String]()
          data.put(t.user, t.url)
          val indexRequest = Requests
            .indexRequest()
            .index("clicks")
            .`type`("type")
            .source(data)
          requestIndexer.add(indexRequest)
        }
      }
    )
    stream.addSink(esBuilder.build())
    env.execute()
  }
}
```

### 输出到 MySQL（JDBC）
需要导入依赖
```
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
```
```
package day4

import day2.Event
import org.apache.flink.connector.jdbc.{JdbcConnectionOptions, JdbcSink, JdbcStatementBuilder}
import org.apache.flink.streaming.api.scala._

import java.sql.PreparedStatement

/**
 * 生成数据addSink推送到mysql
 */
object SinkToMySQLTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)
    val stream = env.fromElements(
      Event("Mary", "./home", 1000L),
      Event("Bob", "./cart", 2000L),
      Event("Alice", "./prod?id=100", 3000L),
      Event("Alice", "./prod?id=200", 3500L),
      Event("Bob", "./prod?id=2", 2500L),
      Event("Alice", "./prod?id=300", 3600L),
      Event("Bob", "./home", 3000L),
      Event("Bob", "./prod?id=1", 2300L),
      Event("Bob", "./prod?id=3", 3300L)
    )
    stream.addSink(
      JdbcSink.sink(
        "INSERT INTO clicks (user, url) VALUES (?, ?)",
        new JdbcStatementBuilder[Event] {
          override def accept(t: PreparedStatement, u: Event): Unit = {
            t.setString(1, u.user)
            t.setString(2, u.url)
          }
        },
        new
            JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
          .withUrl("jdbc:mysql://localhost:3306/test")
          .withDriverName("com.mysql.jdbc.Driver")
          .withUsername("username")
          .withPassword("password")
          .build()
      )
    )
    env.execute()
  }
}
```

### 自定义 Sink 输出
```
与 Source 类似，Flink 为我们提供了通用的 SinkFunction 接口和对应的 RichSinkFunction
抽象类，只要实现它，通过简单地调用 DataStream 的 addSink()方法就可以自定义写入任何外
部存储。
```