---
title: MapReduce的OutputFormat输出十
date: 2022-08-01 16:23:15
tags: [大数据,hadoop,MapReduce]
---
# OutputFormat
`OutputFormat`是`MapReduce`输出的基础类,
所有实现`MapReduce`输出都实现了`OutputFormat`接口

## OutputFormat实现类
<!--more-->

## 默认输出格式
```
TextOutputFormat,按行输出
```

## 自定义OutputFormat
```
例如存储到HBASE, es等
```

## 自定义参数
```
job.setOutputFormatclass(对应的job中去);
```