---
title: hadoop的启动和关闭三
date: 2022-08-01 14:38:31
tags: [大数据,hadoop]
---
# hadoop集群的启动和关闭

## 每台机器手动开启关闭一个角色进程
1. HDFS集群
```
hdfs --daemon start namenode|datenode|secondarynamenode 

hdfs --daemon stop namenode|datenode|secondarynamenode
```

2.YARN集群
```
yarn --daemon start resourcemanager|nodemanager

yarn --daemon stop resourcemanager|nodemanager
```

<!--more-->

## 批量操作开关集群
shell脚本一键启停  
在主角色安装的机子上操作  
在sbin路径下有这个脚本  
前提: 配置好机器之间的SSH免密登录及其workers文件.
```
HDFS集群: 
   start-dfs.sh
   stop-dfs.sh
 
yarn集群:
    start-yarn.sh
    stop-yarn.sh
    
hadoop集群:
     start-all.sh
     stop-all.sh
```
## 查看日志
日志路径 :安装Hadoop路径下 的 /logs


## 查看集群可视化页面

1. HDFS集群
```
地址:http://namenode_host:9870
地址是namenode运行所在及其的主机名或者ip

```

2. yarn集群
```
地址: http://resourcemanager_host:8088
地址是resourcemanager运行所在及其的主机名或者ip

```

