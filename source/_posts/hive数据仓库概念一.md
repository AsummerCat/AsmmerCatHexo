---
title: hive数据仓库概念一
date: 2022-08-01 14:57:31
tags: [大数据,hadoop,hive]
---
# hive数据仓库概念

## 数据仓库概念

数据仓库的输入方式各种各样的数据源,  
最终的输出用于企业的数据分析,数据挖掘,数据报表等方向

<!--more-->

## 数据仓库分层
按照数据流入流程的过程,数据仓库架构可分为三层

源数据(OBS) , 数据仓库(DW) , 数据应用(APP)

数据仓库的数据偏向于主题的概念,
常规的数据库的数据偏向于业务;

# hive的概念
hive是一个数据仓库工具,

1.可以将结构化的数据文件映射为一张数据库表,并提供类似SQL查询功能.
2.hive能够存储很大的数据集,可以直接访问存储在HDFS或者HBase中的文件.
3.可以支持 MapReduce,Spark,Tez这三种分布式计算引擎
