---
title: Flink的CDC案例实操
date: 2022-09-19 17:04:03
tags: [大数据,Flink]
---
# Flink的CDC案例实操

有两种模式选择
1. DataStream方式的应用
2. FlinkSql方式的应用

<!--more-->

## 导入依赖
```
        <dependency>
            <groupId>org.ververica</groupId>
            <artifactId>flink-connector-mysql-cdc</artifactId>
            <version>2.0.0</version>
        </dependency>
```
## DataStream方式的应用
提供了专门的`MySqlSource`
```
  MySqlSource<String> mySqlSource = MySqlSource.<String>builder()
            .hostname("yourHostname")
            .port(yourPort)
            .databaseList("yourDatabaseName") // set captured database
            .tableList("yourDatabaseName.yourTableName") //监控表
            .username("yourUsername")
            .password("yourPassword")
            .deserializer(new JsonDebeziumDeserializationSchema()) // 反序列化方式,可以自定义
            .build();

    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

    // enable checkpoint
    env.enableCheckpointing(3000);

    env
      .fromSource(mySqlSource, WatermarkStrategy.noWatermarks(), "MySQL Source")
      // set 4 parallel source tasks
      .setParallelism(4)
      .print().setParallelism(1); // use parallelism 1 for sink to keep message ordering

    env.execute("Print MySQL Snapshot + Binlog");
```


## FlinkSql方式的应用
```
  tableEnv.executeSql("""
CREATE TABLE MyTable (
 id BIGINT,
 name STRING,
 age INT,
 status BOOLEAN,
 PRIMARY KEY (id) NOT ENFORCED
) WITH (
 'connector' = 'mysql-cdc',
 'url' = 'jdbc:mysql://localhost:3306/mydatabase',
 'table-name' = 'users'
);
    """)
```
差异点: connector方式由`jdbc`修改为`mysql-cdc`

# 