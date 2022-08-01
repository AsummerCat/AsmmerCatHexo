---
title: hadoop调优手册之HDFS压测
date: 2022-08-01 16:27:53
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之HDFS压测

执行命令:
这个是专门用来压测的

### 写测试
```
hadoop jar /hadoop安装路径/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -write -nrFiles 10 -filesize 128MB
```
命令意思:
`-nrFiles` 表示 开启MapTask任务数量,
写入MapTask任务 ,每个文件大小128MB

<!--more-->


### 读测试
```
hadoop jar /hadoop安装路径/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -read -nrFiles 10 -filesize 128MB
```

## 删除测试数据

```
hadoop jar /hadoop安装路径/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-3.1.3-tests.jar TestDFSIO -clean

```