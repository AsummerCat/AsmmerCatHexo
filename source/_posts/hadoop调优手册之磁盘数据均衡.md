---
title: hadoop调优手册之磁盘数据均衡
date: 2022-08-01 16:31:01
tags: [大数据,hadoop,hadoop调优手册]
---
# hadoop调优手册之磁盘数据均衡

## 集群的数据均衡-磁盘间数据均衡
Hadoop3.x新特性
```
由于磁盘空间不足,新增磁盘,
新磁盘无数据, 可以执行磁盘数据均衡命令
注意: 这里是针对单节点里的多个磁盘数据均衡处理的
```

(1)生产均衡计划 (只有一块磁盘,不会生成计划)
```
hdfs diskbalancer -plan hadoop103
```
`hadoop103`是我们定义的文件名称

(2) 执行均衡计划
```
hdfs diskbalancer -execute hadoop103.plan.json
```

(3) 查看当前均衡任务的执行情况
```
hdfs diskbalancer -query hadoop103
```
<!--more-->
(4)取消均衡任务
```
hdfs diskbalancer -cancel hadoop103.plan.json
```