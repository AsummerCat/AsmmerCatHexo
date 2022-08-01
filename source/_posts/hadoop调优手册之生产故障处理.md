---
title: hadoop调优手册之生产故障处理
date: 2022-08-01 16:31:37
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之生产故障处理

## NameNode挂了,并且数据丢失
模拟:
(1) kill -9 `NameNode`进程
(2)删除`NameNode`存储的数据
```
/hadoop目录/data/tmp/dfs/name
```

解决:
(1) 拷贝 `SecondaryNameNode`中的数据到原`NameNode`存储数据目录
<!--more-->
(2) 重启NameNode

## 集群安全模式

配置文件地址:
`hdfs-default.xml`

安全模式:文件系统只接收读数据请求,而不接收删除,修改等变更请求

(1).什么时候会处于安全模式:
-> `NameNode`在加载镜像文件和编辑日志期间,处于安全模式

->`NameNode`在接收`DataNode`注册时,处于安全模式

(2). 退出安全模式
```
//最小可用datanode数量,默认0
dfs.namenode.safemode.min.datanodes:0

//副本数达到最小要求的block 占系统总block数的百分比,默认为0.999f(只允许丢一个块)
dfs.namenode.safemode.threshold-pct:0.999f


//稳定时间,默认值30000毫秒,即30秒
dfs.namenode.safemode.extension:30000
```

(3).基本语法
```
查看安全模式状态
/bin/hdfs dfsadmin -safemode get

进入安全模式状态
/bin/hdfs dfsadmin -safemode enter

离开安全模式状态
/bin/hdfs dfsadmin -safemode leave

等待安全模式状态
/bin/hdfs dfsadmin -safemode wait
```

## 慢磁盘监控
快速找出 响应慢的磁盘or老化的磁盘

1.使用`fio`命令,测试磁盘读写性能
安装依赖
```
sudo yum install -y fio
```
(1).顺序读测试
```
sudo fio -filename=/home/test.log -direct=1 -iodepth 1 -thred rw=read -ioengine=psync -bs=16k -size=2G -numjobs=10 -runtime=60 -group_reporting -name=test-r
```
(2).顺序写测试
```
sudo fio -filename=/home/test.log -direct=1 -iodepth 1 -thred rw=write -ioengine=psync -bs=16k -size=2G -numjobs=10 -runtime=60 -group_reporting -name=test-r
```

(3).随机写测试
```
sudo fio -filename=/home/test.log -direct=1 -iodepth 1 -thred rw=randwrite -ioengine=psync -bs=16k -size=2G -numjobs=10 -runtime=60 -group_reporting -name=test-randw
```

(4).混合随机读写测试
```
sudo fio -filename=/home/test.log -direct=1 -iodepth 1 -thred rw=randwr -ioengine=psync -bs=16k -size=2G -numjobs=10 -runtime=60 -group_reporting -name=test-randw
```