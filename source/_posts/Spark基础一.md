---
title: Spark基础一
date: 2022-08-31 14:02:03
tags: [大数据,Spark]
---
# Spark基础

## Spark下载
https://spark.apache.org/downloads.html
https://archive.apache.org/dist/spark/spark-3.0.0/
ps: scala2.1 对应 spark3.0
## 核心模块

#### SparkSQL
Spark SQL 是 Spark 用来操作结构化数据的组件。通过 Spark SQL ，用户可以使用 SQL或者 Apache Hive 版本的 SQL 方言（ HQL ）来查询数据；
<!--more-->

#### Spark Streaming
Spark Streaming是Spark的基于时间窗口的流式计算组件，支持从kafka、HDFS、Flume等多种数据源获取数据，然后利用Spark计算引擎以微批的形式进行处理，并将结果写入Radis、Kafka、HDFS等系统。

#### Spark MLlib
Spark的机器学习库，提供了统计、分类、回归、预测、推荐等机器学习算法，并高度封装。从而为开发人员提供简单易用的机器学习能力

#### Spark GraphX
GraphX是Spark的分布式图计算组件，利用Pergel提供的API，为开发人员提供可快速实现的图计算能力。

#### Apache Spark Core
Spark Core是Spark的核心，各类核心组件都依赖于Spark Core。
