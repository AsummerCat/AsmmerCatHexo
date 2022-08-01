---
title: Hdoop问题处理
date: 2022-08-01 14:36:49
tags: [大数据,hadoop]
---

# Hdoop问题处理

1.万一集群被误删了文件,导致nameNode无法启动
```
首先要kill 所有hadoop进程,
然后删除集群下的 /data /logs 这两个目录, (删除历史数据)
然后执行

hdfs namenode -format

```
<!--more-->

