---
title: SparkSql的作用一
date: 2022-08-31 14:27:56
tags: [大数据,Spark,SparkSql]
---
# SparkSql的作用

等价: sparkSQL   与 hive ON spark


1.无缝的整合了 SQL 查询和 Spark 编程
2.使用相同的方式连接不同的数据源
3.在已有的仓库上直接运行 SQL 或者 HiveQL
4.通过 JDBC 或者 ODBC 来连接
<!--more-->

## DataFrame 是什么
在 Spark 中，DataFrame 是一种以 RDD 为基础的分布式数据集，类似于传统数据库中
的二维表格。DataFrame 与 RDD 的主要区别在于，前者带有 schema 元信息，即 DataFrame
所表示的二维表数据集的每一列都带有名称和类型。这使得 Spark SQL 得以洞察更多的结构
信息，从而对藏于 DataFrame 背后的数据源以及作用于 DataFrame 之上的变换进行了针对性
的优化，最终达到大幅提升运行时效率的目标。反观 RDD，由于无从得知所存数据元素的
具体内部结构，Spark Core 只能在 stage 层面进行简单、通用的流水线优化。

![image](/img/2022-08-29/1.png)

## DataSet 是什么
DataSet 是分布式数据集合。DataSet 是 Spark 1.6 中添加的一个新抽象，是 DataFrame
的一个扩展。它提供了 RDD 的优势（强类型，使用强大的 lambda 函数的能力）以及 Spark
SQL 优化执行引擎的优点。DataSet 也可以使用功能性的转换（操作 map，flatMap，filter
等等）。
