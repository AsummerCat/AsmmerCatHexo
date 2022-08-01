---
title: redis主从同步读写分离
date: 2018-11-14 14:37:15
tags: [redis]
---

# Redis主从同步
Redis支持主从同步。数据可以从主服务器向任意数量的从服务器上同步，同步使用的是发布/订阅机制。

# 配置主从同步

Mater Slave的模式，从Slave向Master发起SYNC命令。

可以是1 Master 多Slave，可以分层，Slave下可以再接Slave，可扩展成树状结构。

<!--more-->

## 配置Mater，Slave

配置非常简单，只需在slave的设定文件中指定master的ip和port


### <font color="red">Master： test166</font>

修改设定文件，服务绑定到ip上

```
# vi /etc/redis.conf
bind 10.86.255.166
```

重启Redis

```
 systemctl restart redis
```

### <font color="red">Slave： test167</font>

修改设定文件，指定Master

```
slaveof <masterip> <masterport>    指定master的ip和port
masterauth <master-password>       master有验证的情况下
slave-read-only yes                设置slave为只读模式
```

也可以用命令行设定：

```
redis 127.0.0.1:9999> slaveof localhost 6379
OK
```

## 同期情况确认

### Master：

```
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=10.86.255.167,port=6379,state=online,offset=309,lag=1
……
```

###  Slave：

```
127.0.0.1:6379> INFO replication
# Replication
role:slave
master_host:10.86.255.166
master_port:6379
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:365
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

同期正常时：

master_link_status:up

master_repl_offset和slave_repl_offset相等，

master_last_io_seconds_ago在10秒内。

---

# Slave升级为Master

Master不可用的情况下，停止Master，将Slave的设定无效化后，Slave升级为Master

```
redis 127.0.0.1:9999> SLAVEOF NO ONE
 OK
 
redis 127.0.0.1:9999> info
......
role:master
......
```

# Health Check 心跳检测

Slave按照repl-ping-slave-period的间隔（默认10秒），向Master发送ping。

如果主从间的链接中断后，再次连接的时候，2.8以前按照full sync再同期。2.8以后，因为有backlog的设定，backlog存在master的内存里，重新连接之前，如果redis没有重启，并且offset在backlog保存的范围内，可以实现从断开地方同期，不符合这个条件，还是full sync

 

用monitor命令，可以看到slave在发送ping


```
127.0.0.1:6379> monitor
OK
1448515184.249169 [0 10.86.255.166:6379] "PING"
```

#  设置Master的写行为

2.8以后，可以在设定文件中设置，Master只有当有N个Slave处于连接状态时，接受写操作


```
min-slaves-to-write 3
min-slaves-max-lag 10

```

