---
title: Flink的CDC种类区别
date: 2022-09-19 17:02:48
tags: [大数据,Flink]
---
# Flink的CDC种类区别

## CDC是啥

CDC 是 Change Data Capture(变更数据获取)的简称,,
核心思想是:检测并捕获数据库的变动(包括数据或数据表的插入,更新以及删除等),将这些变更按照发生的顺序完整记录下来,写入到消息中间件中以供其他服务进行订阅和消费

## CDC的模式
主要分为`基于查询`和基于`Binlog` 两种模式
<!--more-->
`基于查询`的有: Sqoop (批处理)
`基于Binlog`的有: Maxwell,Canal (流处理)

## Flink的CDC是啥
这个是一个可以直接从mysql或者pgSql等数据库直接读取全量数据和变更数据的source组件,也是基于binlog

`flink-cdc-connectors`
