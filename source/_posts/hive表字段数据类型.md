---
title: hive表字段数据类型
date: 2022-08-01 14:55:41
tags: [大数据,hadoop,hive]
---
![](/img/2022-08-01/23.png)
跟mysql对比 多了 STRING  字符串变长;

<!--more-->
# hive文件存储格式
hive支持的存储数据格式主要有:
`TEXTFILE`,`SEQUENCEFILE`, `ORC`,`PARQUET`

其中:
`TEXTFILE`,`SEQUENCEFILE` 按行存储,
`ORC`,`PARQUET` 按列存储

## `TEXTFILE`,`SEQUENCEFILE` 按行存储


<!--more-->

## TextFile格式
默认格式,数据不做压缩,磁盘开销大,数据解析开销大.可以结合压缩`Gzip`,`Bzip2`使用,但使用`Gzip`这种方式,hive不会对数据进行切分,从而无法对数据进行并行操作;

## Orc格式
是hive.0.11引入的新存储格式

## Parquet格式
Parquet是二进制存储,所以是不可以直接读取的,文件中包裹该文件的数据和元数据,<font color='red'>因此Parquet文件是自解析的</font>


