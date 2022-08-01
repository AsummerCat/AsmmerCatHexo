---
title: hadoop调优手册之HDFS存储方式修改冷热数据分离
date: 2022-08-01 16:30:08
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之冷热数据分离
异构存储主要解决,不同的数据,存储在不同类型的硬盘中,达到最佳性能的问题

## 存储类型
1.RAM_DISK (内存镜像文件系统)
2.SSD (SSD固态硬盘)
3.DISK (普通磁盘,在HDFS中,如果没有主动声明数据目录存储类型默认都是DISK)
4.ARCHIVE (没有特指哪种存储介质,主要是计算能力差,容量大的存储介质,用来解决数据量的容量扩容问题,一般用于归档)

## 存储策略
<font color='red'>说明:从Lazy_persist到Cold,分别代表了设备的访问数据从快到慢</font>


| 存储ID | 存储名称 | 副本分布 | 备注 |
| --- | --- | --- | --- |
| 15 | Lazy_Persist | RAM_DISK:1 ,DISK:n-1| 一个副本保存在内存RAM_DISK中,其余副本保存在磁盘中 |
| 12 | ALL_SSD |SSD:n  | 所有副本都保存在SSD中 |
|  10| One_SSD | SSD:1 DISK:n-1 | 一个副本保存在SSD中,其余副本保存在磁盘中 |
| 7 | Hot(默认) |DISK:n  | 所有副本保存在磁盘中,这也是默认的存储策略 |
| 5 | Warm | DISK:1,ARCHIVE:n-1 | 一个副本保存在磁盘上,其余副本保存在归档存储上 |
| 2 | Cold | ARCHIVE:n |  所有副本都保存在归档存储上|

 <!--more-->

## 异构存储shell操作

#### 1.查看当前有那些存储策略可以用
```
hdfs storagepolicies -listPolicies
```

#### 2.为指定路径(数据存储目录)设定指定的存储策略
```
hdfs storagepolicies -setStoragePolicy -path xxxx路径 xxx存储策略 
```

#### 3.获取指定路径(数据存储目录或文件)的存储策略
```
hdfs storagepolicies -getStoragePolicy -path xxxx路径
```

#### 4.取消存储策略;执行该命令之后该目录或文件,以其上级的目录为准,如果是根目录,那么就是HOT
```
hdfs storagepolicies -unsetStoragePolicy -path xxx路径
```

#### 5.查看文件快的分布
```
/bin/jdfs fsck xxx路径 -files -blocks -locations
```

#### 6.查看集群节点
```
hadoop dfsadmin -report
```

#### 7.手动转移文件块
```
hdfs mover /hdfsdata
```

#### 8.如果存储在RAM_DISK中要修改配置 不推荐使用
```
<!--是否开启存储在内存中 ,默认大小为0,如果要用的话最大值为大小-->
dfs.datanode.max.locked.memory

<!--hdfs的块大小-->
dfs.block.size

```


## 如何开启副本存储方式的变更

1.修改每个节点的`hdfs-site.xml`添加如下信息
```
<!-当前节点开启两个副本-->
<property>
  <name>dfs.replication</name>
  <value>2</value>
</property>

<!-是否开启存储策略,默认开启-->
<property>
  <name>dfs.storage.policyt.enabled</name>
  <value>true</value>
</property>

<!-- 强行指定存储介质-->
<property>
  <name>dfs.datanode.data.dir</name>
  <value>[SSD]file:xxxx/ssd,[RAM_DISK]file:xxxx/ram_disk</value>
</property>
```
注意1:每台机子都不一样,需要分别指定

注意2: [RAM_DISK][SSD]用来标注存储介质,这边强行指定第一个副本为SSD,第二个为RAM_DISK

注意3:`hdfs-site.xml`中有个配置的`dfs.datanode.data.dir`参数,现在使用的强行指定介质存储,所有要移除掉这个参数



