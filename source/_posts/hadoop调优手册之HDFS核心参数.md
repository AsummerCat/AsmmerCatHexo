---
title: hadoop调优手册之HDFS核心参数
date: 2022-08-01 16:26:34
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之HDFS核心参数

## NameNode生产配置
#### 1.NameNode内存计算配置
每个文件块大概占用150byte,一台服务器128G内存为例,能存储多少文件块
```
128*1024*1024*1024/150byte =9.1亿个
```
1. hadoop2.x版本,配置NameNode内存
```
NameNode内存默认2000m,
如果服务器内存4g,NameNode可配置3g

在hadoop-env.sh文件中配置

HADOOP_NAMENODE_OPS=-Xmx3072m
```
<!--more-->
2.Hadoop3.X版本,配置NameNode内存
```
hadoop-env.sh 中是动态分配的
需要设置堆栈内存大小,如果未处理,
则按照机器内存配置

export HADOOP_HEAPSIZE_MIN

export HADOOP_HEAPSIZE_MAX
```
可参考CDH的配置:
```
NameNode最小值1G,每增加100000个block,增加1G内存

DataNode最小值4G,block数,或者副本数升高,都应该调大DataNode的值

一个DataNode上的副本总数低于4000000,调为4G,
超过4000000每增加1000000,增加1G
```
具体修改内容:
```
export HDFS_NAMENODE_OPTS=" -Xmx1024m"

export HDFS_DATANODE_OPTS=" -Xmx1024m"
```

#### 2.NameNode心跳并发配置
`hdfs-site.xml`
```
<!--NameNode 有一个工作线程池,用来处理不通DataNode的并发心跳及其客户端并发的元数据操作 -->
<!--对于大集群或者有大量客户端的集群来说,通常需要增大该参数. 默认值是10-->
<property>
  <name>dfs.namenode.hadler.count</name>
  <value>21</value>
</property>
```
企业经验: dfs.namenode.handler.count=20*log(cluster size) =21
也就是集群规模(DataNode台数)为3台时,该配置为21

=20*math.log(3)

#### 开启HDFS的回收站功能
配置位置:`core-site.xml`

可以将删除的文件在不超时的情况下,恢复原数据,起到防止误删除,备份等作用
1.配置参数详情 core-site.xml
```
<!--开启并设置文件存活时间,
默认值0表示禁用回收站,其他值表示设置文件的存活时间(单位分钟)-->
fs.trash.interval=60

<!--检查回收站的间隔时间 如果等于0的话 等同于文件存活时间 (单位分钟)-->
fs.trash.checkpoint.interval=10
```
要求`fs.trash.checkpoint.interval`<=`fs.trash.interval`

2. 开启回收站
```
<property>
  <name>fs.trash.interval</name>
  <value>1</value>
</property>
```

3. 查看回收站
   回收站目录: HDFS集群中的路径 /user/xxx/.Trash/xxx
```
1.通过网页删除的不会走回收站

2.通过程序删除的 需要手动调用`moverToTrash(path)`,
不然不经过回收站
Trash trash=new Trash(conf);
trash.moverToTrash(path);

3.只有在命令行操作,才会进去回收站
Hadoop fs -rm -r xxxx/路径
```

4. 恢复回收站数据
```
hadoop fs -mv /user/xxx/.Trash/xxx /恢复到的路径
```