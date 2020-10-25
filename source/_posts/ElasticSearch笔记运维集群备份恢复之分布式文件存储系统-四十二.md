---
title: ElasticSearch笔记运维集群备份恢复之分布式文件存储系统(四十二)
date: 2020-09-08 00:00:03
tags: [elasticSearch笔记]
---

# hadoop hdfs分布式文件存储系统 (推荐)

## snapshot+ hdfs进行数据备份

replica没法进行容灾性的数据保护,比如所有机子宕机等情况.
这样我们就需要对集群中所有的数据进行备份了

<!--more-->

`snapshot api`进行备份的话  
->  这个api会将集群当前的状态和数据全部存储到一个外部的共享目录中去,如NAS,或者hdfs.  
->   第一次会备份全量的数据,但是接下来的`snapshot`就会备份增量数据了

### 1.创建备份仓库
```
PUT _snapshot/my_backup
{
    "type": "fs",
    "settings":{
        "location":"/mount/backups/my_backup"
    }
}

这里用了shard filesystem作为仓库类型,
包括了仓库名称以及仓库类型是fs,还有仓库的地址
还有一些参数可以配置:
'max_snapshot_per_sec': 默认20mb/s
这个参数用于指定数据从es灌入仓库的时候,进行限流

'max_restore_bytes_per_sec': 默认20mb/s
这个参数用于指定数据从仓库中会恢复到es的时候,进行限流
如果网络是非常快速的,那么可以提高这两个参数的值,
可以加快每次备份和恢复的数据

```
比如:
```
PUT _snapshot/my_backup
{
    "type": "fs",
    "settings":{
        "location":"/mount/backups/my_backup",
        # es导出
        'max_snapshot_per_sec': "50mb",
        # es导入
        'max_restore_bytes_per_sec':'50mb'
    }
}

```
## 基于hdfs创建仓库
需要给es安装 repository-hdfs的插件  
`./elasticsearch-plugin install repository-hdfs`  
必须在每个节点上都安装,然后重启整个集群

注意:  
可能会产生`hdfs`版本不兼容 如果不兼容的话
->考虑在`hdfs plugin`的文件夹里,将hadoop相关jar包都替换我们自己的hadoop版本对应的jar.及时hadoop已经在es所在机器上也安装了,但是为了安全考虑,还是应该将hadoop jar放在hadfs plugin的目录下   

安装`repository-hdfs`插件后,然后就可以创建仓库了  
```
curl -XGET 'http:localhost:9200/_count?pretty' -d '
{
    "query":{
        "match_all":{}
    }
}

```
```
curl -XPUT 'http://elasticsearch:9200/_snapshot/my_hdfs_repository' -d'
{
    "type":"hdfs",
    "settings":{
        "uri":'hdfs://elasticsearch:9200',
        "path":"elasticsearch/repository/my_hdfs_repository",
        "max_snapshot_bytes_per_sec":"50mb",
        "max_restore_bytes_per_sec":"50mb"
        
    }
}

```

# 创建备份
## 对所有open的索引进行snapshotting备份
```
后台运行:
'PUT shapshot/my_hdfs_repository/snapshot 1 '
这条命令就会将所有open的说明都放入一个叫做snapshot 1的备份,并且放入'my_backup'仓库中


前台运行:
'PUT shapshot/my_hdfs_repository/snapshot 1?wait_for_completion=true'

```

## 对指定的索引进行snapshotting备份
默认的备份是会备份所有的索引,但是有的时候不需要备份这么多不重要的索引
```
PUT _snapshot/my_backup/snapshot_2
{
    "indices":"index_1,index2",
    "ignore_unavailable":true,
    "include_global_state":false,
    "partial":true
}

"ignore_unavailable":
 忽略不存在的索引,
 如果不设置为true,那么某个索引丢失了,备份失败

"include_global_state":
是否开启cluster的全局state作为snapshot的一部分备份

"partial":
默认情况下,某个索引的部分primary shard不可用那么备份会失败,
开启partial 则继续备份


```
注意: 备份的过程都是增量进行的

## 查看备份列表
一旦有备份文件入库后,我们可以查看备份
```
指定备份:
_snapshot/my_backup/snapshot_2

全部备份:
GET _snapshot/my_backup/_all

```


## 删除snapshot备份

注意第一要用api删除,不能直接hdfs删除.  
因为snapshot是增量的,很多需要依赖旧数据
```
delete _snapshot/my_backup/snapshot_2

```

## 监控snapshottiing的进度
```

查看备份的进度

GET _snapshot/my_backup/snapshot_2/_status



```

## 取消snapshot备份过程
直接删除就可以了
```
delete _snapshot/my_backup/snapshot_2

```


# 备份的数据恢复
```
后台运行:
post  _snapshot/my_backup/snapshot_2/_restore


前台运行:
post _snapshot/my_backup/snapshot_2/_restore?wait_for_completion=true
```

## 带参数的数据恢复
```

post  _snapshot/my_backup/snapshot_2/_restore
{
    //恢复指定索引
    "indices":"index_1",
    //跟备份的参数意思类似 
    "igore_unavailable": true,
    //跟备份的参数意思类似 
    "include_global_state": true,
    //恢复->符合正则表达式的索引
    "rename_pattern": "index_(.+)",
    //恢复索引的时候指定为->重命名索引
    "rename_replacement": "restored_index_$1"
}

```

## 监控恢复进度
```

GET  my_index/_recovery?pretty
```