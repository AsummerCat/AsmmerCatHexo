---
title: hadoop调优手册之HDFS一键扩容和缩容
date: 2022-08-01 16:29:37
tags: [大数据,hadoop,hdfs,hadoop调优手册]
---
# hadoop调优手册之HDFS一键扩容和缩容


## 添加白名单
白名单: 表示在白名单里的IP,可以存储数据

配置如下:
在`NameNode`节点的`/hadoop安装路径/etc/hadoop/`目录下分别创建`whiteist`,`blacklist`文件

#### 1.创建白名单:
`vim blacklist`

<!--more-->
直接加入对应的ip 或者域名
```
192.168.2.2
192.168.2.2
```
#### 2.在`hdfs-site.xml`配置文件中增加`dfs.hosts`配置参数
```
<!--白名单-->
<property>
    <name>dfs.hosts</name>
    <value>/hadoop安装路径/etc/hadoop/whitelist</value>
</property>

<!--黑名单-->
<property>
    <name>dfs.hosts.exclude</name>
    <value>/hadoop安装路径/etc/hadoop/blacklistlist</value>
</property>
```

#### 3.第一次添加白名单必须重启集群,后面更新的话,只要刷新`NameNode`节点即可

刷新`NameNode`节点
```
hdfs dfsadmin -refreshNodes
```


## 新增数据节点
在原有集群基础上动态添加新的数据节点

```


```