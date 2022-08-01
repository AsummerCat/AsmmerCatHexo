---
title: hdfs的存储策略三
date: 2022-08-01 14:42:06
tags: [大数据,hadoop,hdfs]
---
# hdfs的存储策略

## HDFS存储策略
可以选择存储在
```
mem 内存             非持久化
ssd  固态硬盘        持久化
hard disk 机械硬盘   持久化 

```
注意需要手动手动进行配置,因为HDFS无法自动识别磁盘属性  
需要在 配置属性时主动声明.HDFS并没有自动检测的能力
<!--more-->

## 参数配置
```

如果目录前没有带上[SSD] [DISK] [ARCHIVE] [RAM_DISK]
这4种类型中的任意一种,则默认是DISK类型

```

## 配置存储介质类型
```
hdfs-site.xml

<property>
  <name>dfs.datanode.data.dir</name>
  <value>[DISK]file://${hadoop.tmp.dir}/dfs/data,[SSD]file://${hadoop.tmp.dir}/dfs/data/SSD</value>
</property>

以逗号隔开
```