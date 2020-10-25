---
title: ElasticSearch笔记运维集群节点之间优雅启停(四十一)
date: 2020-09-06 22:010:17
tags: [elasticSearch笔记]
---

# 以后台模式运行 

```
./bin/elasticsearch -d -p pid
```
上面的命令中`-d option`用来指定es以`daemon`进程方式启动,
并且`-p option`指定将进程id近路在指定文件中

此外,启动es进程的时候,还开业直接覆盖一些配置,是用`-E`即可 ,方便调试
```
./bin/elasticsearch -d  -Epath.conf=/etc/elasticsearch

```
<!--more-->

# 停止es

```
jps | grep Elasticsearch

kill -SIGTERM 15516

```