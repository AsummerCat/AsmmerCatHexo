---
title: ElasticSearch笔记运维之升级方案(四十三)
date: 2020-09-09 08:44:47
tags: [ElasticSearch笔记]
---

# es版本的升级

## es版本升级的通用步骤
1. 看一下最新的版本的changes 文档,每个小版本之间的升级,都有哪些变化,新功能,bugfix
2. 在开发环境下的机器中,先实验一下版本的升级,定一下升级的步骤和操作方案的文档
3. 对数据进行全量的备份->最次的情况 升级失败,还能安装回原来版本
4. 检查升级之后plugin是否跟es主版本兼容,升级完es之后,还要重新安装一下你的plugin


<!--more-->

## es不同版本升级的策略
1. es 1.x 升级到5.x ,是需要用`索引重建策略`的 -> 跨两个版本升级
2. es 2.x 升级到5.x,是需要用`集群重启策略`的 ->跨单个版本
3. es 5.x 升级为5.y,是需要用`节点依次重启策略的` ->小版本升级

需要注意的是 :  
es的每个大版本都可以读取上一个大版本创建的索引文件,但是如果是上上个大版本创建的索引,是不可以读取的

# 小版本升级之节点依次重启策略
 `rolling upgrade`(节点依次重启策略)

这种升级方案: 会让es集群每次升级一个node,对终端用户来说,是没有停机时间的.在一个es集群中运行多个版本,长时间的话是不行的,因为shard是没法从一较新版本的node上replicate到较旧版本的node上的


## (1)关闭自动恢复节点的功能
因为关闭一个`node`之后,这个node上的`shard`全都不可用了,此时`shard allocation`机制会等待1分钟,开始`shard recovery`过程,也就是将丢失的`primary shard`的`replica shard`提升为`primary shard `,同时创建更多的`replica shard` 满足副本数量,但是这个过程会导致大量的IO操作,是没有必要的.  
因此在开始升级一个`node`,以及关闭这个`node`之前,先进制`shard allocation`机制
```
PUT _cluster/settings
{
    "transient":{
        "cluster.routing.allocation.enable":"none"
    }
}

```
## (2)停止非核心业务的写入操作,以及执行一次flush操作
保证输入已经入es的磁盘中  
执行以下命令使其刷盘
```
POST _flush/synced

```
## (3)停止一个node 然后升级这个node

## (4)升级plugin

## (5)启动es node

## (6)然后重新开启shard allocation 节点恢复
```
PUT _cluster/settings
{
    "transient":{
        "cluster.routing.allocation.enable":"all"
    }
}

```
## (7)等待node完成shard recover过程
我们要等待cluster完成shard allocation的过程,可以通过以下命令查看进度
```
GET _cat/health?pretty
```
一定要等待`cluster`的`status`从`yellow`变成`green`才可以,green就以为这所有的`primary shard`和`replica shard`都可以用了

注意:     在这个期间内如果分配了一个高版本的`primary shard`,是一定不会将其`replica`复制给较旧的版本的,因为新版本数据格式跟旧版本的是不兼容的  
那么有些`replica shard`就会是`unassgied`状态,此时`cluster status`就会保持`yellow`,等升级完成就会变成`green`

## (8)重复以上操作直到所有node都升级完成


# 大版本升级之集群重启策略(跨1版本)


## 跟小版本升级差不多
需要注意的是: 需要将所有node都关闭之后再升级  
因为如果采用滚动升级的方案,可能导致es集群中的有些节点是高版本,有些是低版本

## (1) 禁止shard allocation

## (2) 执行一次flush操作

## (3) 关闭和升级所有的node

## (4) 升级plugin

## (5) 启动cluster集群

## (6)等待cluster状态变成yellow
让所有节点都被发现之后再去处理`node allocation`

## (7)重新启用allocation


# 大版本升级之 索引重建策略(跨多版本)

使用`upgrading with reindex-from-remote`策略实现跨版本升级

## 用reindex from remote的方式,在两个集群之间迁移index数据
```
POST _reindex
{
    "source":{
        "remote":{
        //低版本的es
            "host":"http://xxx:9200",
            "username":"user",
            "password":"pass"
        },
        //备份的索引
        "index":"source"
    },
    "dest":{
        "index":"source"
    }
}

```
注意: `remote cluster`必须显示在`elasticsearch.yml`中列入白名单,使用`redinx.remote.whitelist`属性

```
低版本上配置 新版本连接的白名单

reindex.remote.whitelist: ["localhost:9200"]
```

reindex过程中会使用默认的`on-heap buffer`最大大小是100mb,如果要迁移的数据量很大,需要将`batch size`设置的很小,这样每次同步的数据就很少,使用`size`属性.还可以设置`socket_timeout`和`connect_timeout`,比如下面:
```
POST _reindex
{
    "source":{
        "remote":{
        //低版本的es
            "host":"http://xxx:9200",
            "username":"user",
            "password":"pass",
            "socket_timeout":"1m",
            "connect_timeout":"10s"
        },
        "index":"source",
        "size":10,
        "query":{
            "match":{
                "test":"data"
            }
        }
    },
    "dest":{
        "index":"dest"
    }
}
```