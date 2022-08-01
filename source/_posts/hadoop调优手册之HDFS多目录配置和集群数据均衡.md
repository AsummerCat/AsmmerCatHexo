---
title: hadoop调优手册之HDFS多目录配置和集群数据均衡
date: 2022-08-01 16:28:56
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之HDFS多目录配置和集群数据均衡

## NameNode多目录配置:
1.NameNode的本地目录可以配置成多个,<font color='red'>且每个目录存放内容相同</font>,增加了可靠性

2.具体配置如下:
<!--more-->

(1) 在`hdfs-site.xml`文件中添加如下内容
```
<property>
  <name>dfs.namenode.name.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/name1,file://${hadoop.tmp.dir}/dfs/name2</value>
</property>
```
注意: 因为每台服务器节点的磁盘情况不通,所以这个配置,可以不共享给其他集群主机
<font color='red'>`hadoop.tmp.dir` 这个是在`core-default.xml`里面, 临时数据默认保存一个月后删除</font>


(2)停止集群,删除三台节点的`data`和`logs`中所有数据
```
路径在 hadoop里
rm -rf data/ logs/
rm -rf data/ logs/
rm -rf data/ logs/
```

(3)格式化集群并重启
```
bin/hdfs namenode -format
sbin/start-dfs.sh
```

## DataNode多目录配置:
这里跟`NameNode`有区别;
NameNode多目录是备份副本;
DataNode多目录是多个节点存储数据不一样的(数据不是副本),一起工作;

(1)DataNode可以配置成多个目录,
<font color='red'>每个目录存储的数据不一样</font>(数据不是副本)

(2)具体配置如下:
在`hdfs-site.xml`文件中添加如下内容:
```
<property>
  <name>dfs.datanode.dir</name>
  <value>file://${hadoop.tmp.dir}/dfs/data1,file://${hadoop.tmp.dir}/dfs/data2</value>
</property>
```

