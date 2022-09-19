---
title: Flink的TableAPI和SQL十一
date: 2022-09-19 17:01:59
tags: [大数据,Flink]
---
# Flink的Table API 和 SQL

## 引入依赖
```
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-scala-bridge_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <!--ide运行环境-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
            <scope>provided</scope>
        </dependency>

        <!--本地环境自定义的数据格式来做序列化-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-common</artifactId>
            <version>${flink.version}</version>
        </dependency>
```
<!--more-->

## 基础demo
```
    // 创建表环境
    val tableEnv = ...;
    // 创建输入表，连接外部系统读取数据
    tableEnv.executeSql("CREATE TEMPORARY TABLE inputTable ... WITH ( 'connector' = ... )")
    // 注册一个表，连接到外部系统，用于输出
    tableEnv.executeSql("CREATE TEMPORARY TABLE outputTable ... WITH ( 'connector' = ... )")
    // 执行 SQL 对表进行查询转换，得到一个新的表
    val table1 = tableEnv.sqlQuery("SELECT ... FROM inputTable... ")
    // 使用 Table API 对表进行查询转换，得到一个新的表
    val table2 = tableEnv.from("inputTable").select(...)
    // 将得到的结果写入输出表
    val tableResult = table1.executeInsert("outputTable")
```
## 基础案例
```
package day8

/**
 * SQL语法编写查询Flink
 */
object SqlTest {
  def main(args: Array[String]): Unit = {
    val eventStream = env
      .fromElements(
        Event("Alice", "./home", 1000L),
        Event("Bob", "./cart", 1000L),
        Event("Alice", "./prod?id=1", 5 * 1000L),
        Event("Cary", "./home", 60 * 1000L),
        Event("Bob", "./prod?id=3", 90 * 1000L),
        Event("Alice", "./prod?id=7", 105 * 1000L)
      )
    // 获取表环境
    val eventStream = StreamTableEnvironment.create(env)
    // 将数据流转换成表
    val eventTable = tableEnv.fromDataStream(eventStream)
    // 用执行 SQL 的方式提取数据
    val visitTable = tableEnv.sqlQuery("select url, user from " + eventTable)
    // 将表转换成数据流，打印输出
    tableEnv.toDataStream(visitTable).print()
    // 执行程序
    env.execute()
  }
}

```

# 连接到外部系统

## 连接kafka
```
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
```
```
package day9

import day2.Event
import org.apache.flink.streaming.api.scala._
import org.apache.flink.table.api.bridge.scala.StreamTableEnvironment

/**
 * SQL语法连接外部的Kafka
 */
object KafkaSqlTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment

    val eventStream = env.fromElements(
      Event("Alice", "./home", 1000L),
      Event("Bob", "./cart", 1000L),
      Event("Alice", "./prod?id=1", 5 * 1000L),
      Event("Cary", "./home", 60 * 1000L),
      Event("Bob", "./prod?id=3", 90 * 1000L),
      Event("Alice", "./prod?id=7", 105 * 1000L)
    )
    // 获取表环境
    val tableEnv = StreamTableEnvironment.create(env)
    // 将数据流转换成表
    val eventTable = tableEnv.fromDataStream(eventStream)

    tableEnv.executeSql(
      """
    CREATE TABLE KafkaTable (
      `user` STRING,
      `url` STRING,
      `ts` TIMESTAMP(3) METADATA FROM 'timestamp'
    ) WITH (
      'connector' = 'kafka',
    'topic' = 'events',
    'properties.bootstrap.servers' = 'localhost:9092',
    'properties.group.id' = 'testGroup',
    'scan.startup.mode' = 'earliest-offset',
    'format' = 'csv'
    )
    """)

    // 将表转换成数据流，打印输出
    tableEnv.toDataStream(eventTable).print()
    // 执行程序
    env.execute()
  }


}

```

## JDBC
类似
```
  package day9

import day2.Event
import org.apache.flink.streaming.api.scala._
import org.apache.flink.table.api.bridge.scala.StreamTableEnvironment

/**
 * SQL语法连接外部的mysql数据库
 */
object JdbcSqlTest {
  def main(args: Array[String]): Unit = {
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val eventStream = env.fromElements(
      Event("Alice", "./home", 1000L),
      Event("Bob", "./cart", 1000L),
      Event("Alice", "./prod?id=1", 5 * 1000L),
      Event("Cary", "./home", 60 * 1000L),
      Event("Bob", "./prod?id=3", 90 * 1000L),
      Event("Alice", "./prod?id=7", 105 * 1000L)
    )
    // 获取表环境
    val tableEnv = StreamTableEnvironment.create(env)
    // 将数据流转换成表
    val eventTable = tableEnv.fromDataStream(eventStream)

    //这里在 WITH 前使用了 PARTITIONED BY 对数据进行了分区操作。文件系统连接器支持
    //对分区文件的访问。
    tableEnv.executeSql(
      """
CREATE TABLE MyTable (
 id BIGINT,
 name STRING,
 age INT,
 status BOOLEAN,
 PRIMARY KEY (id) NOT ENFORCED
) WITH (
 'connector' = 'jdbc',
 'url' = 'jdbc:mysql://localhost:3306/mydatabase',
 'table-name' = 'users'
);
    """)

    // 将表转换成数据流，打印输出
    tableEnv.toDataStream(eventTable).print()
    // 执行程序
    env.execute()
  }


}

```