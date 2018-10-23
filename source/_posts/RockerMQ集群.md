---
title: RockerMQ集群
date: 2018-10-23 11:09:03
tags: [RocketMQ,消息队列]
---

>[RockerMQ介绍 及搭建双master模式](https://www.cnblogs.com/shmilyToHu/p/8875521.html)  
>[RocketMQ笔记(2)_双主双从部署](https://my.oschina.net/xcafe/blog/814135)



# RocketMQ集群方式
推荐的几种 Broker 集群部署方式，这里的Slave 不可写，但可读，类似于 Mysql 主备方式。

<!--more-->

## 单个Master
这种方式风险较大，一旦Broker重启或者宕机时，会导致整个服务不可用，不建议线上环境使用
##  多master模式
一个集群无Slave，全是master，例如2个master或者3个master

　　优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，及时机器宕机不可回复的情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失少量消息，同步刷盘一条不丢）性能最高。

　　缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会收到影响。

　　###先启动NameServer

　　###在机器A，启动第一个Master

　　###在机器B，启动第二个Master

##  多Master多Slave模式，异步复制
每个Master配置一个Slave，有多对Master-Slave，HA采用异步复制方式，主备有短暂消息延迟，毫秒级。

　　优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，因为Master宕机后，消费者仍然可以从Slave消息，此过程对应用透明，不需要人工干预，性能同多Master模式几乎一样。

　　缺点：Master宕机，磁盘损坏情况，会丢失少量消息

　　### 先启动 NameServer

　　### 在机器 A，启动第一个 Master

　　### 在机器 B，启动第二个 Master

　　### 在机器 C，启动第一个 Slave

　　### 在机器 D，启动第二个 Slave


## 多Master多Slave模式，同步双写
每个 Master 配置一个 Slave，有多对Master-Slave，HA 采用同步双写方式，主备都写成功，向应用返回成功。

　　优点：数据与服务都无单点，Master宕机情况下，消息无延迟，服务可用性与 数据可用性都非常高

　　缺点：性能比异步复制模式略低，大约低 10%左右，发送单个消息的 RT 会略高。目前主宕机后，备机不能自动切换为主机，后续会支持自动切换功能 。

　　### 先启动 NameServer

　　### 在机器 A，启动第一个 Master

　　### 在机器 B，启动第二个 Master

　　### 在机器 C，启动第一个 Slave

　　### 在机器 D，启动第二个 Slave

　　以上 Broker 与 Slave 配对是通过指定相同的brokerName 参数来配对，Master 的 BrokerId 必须是 0，Slave 的BrokerId 必须是大与 0 的数。另外一个 Master 下面可以挂载多个 Slave，同一 Master 下的多个 Slave通过指定不同的 BrokerId 来区分。  
　　
<font color="red">以上 Broker 与 Slave 配对是通过指定相同的brokerName 参数来配对，Master 的 BrokerId 必须是 0，Slave 的BrokerId 必须是大与 0 的数。另外一个 Master 下面可以挂载多个 Slave，同一 Master 下的多个 Slave通过指定不同的 BrokerId 来区分。</font>

---

# 集群部署 双master双Slave

## 创建两台虚拟机 分别部署rockermq

## 修改Host信息:
 tips:每台机子都要修改 把集群里面的机子对应的全部复制进去  
 依次修改每台主机的hosts文件：

`vim /etc/hosts`
 
```
添加如下信息：

192.168.0.11 mqnameserver1

192.168.0.14 mqnameserver2

192.168.0.16 mqnameserver3

192.168.0.17 mqnameserver4

192.168.0.11 broker-a

192.168.0.14 broker-b

192.168.0.16 broker-a-s

192.168.0.17 broker-b-s
```

## 主机名修改(非必须 好区分)

```
依次修改每台主机的network文件：

vim /etc/sysconfig/network

根据服务器信息设定主机名：

HOSTNAME=broker-b

这一步非必需，但为了让主机名更加清晰，所以重新命名。
```

## Broker配置文件 
  这里可以配置自己的配置文件 不用修改原来的   
  启动borker的时候 带上参数就可以了
  
  ```
  sh mqbroker -c ../conf/me-2m-2s-async/broker.p
  ```

### 创建配置文件模板(也可不创建)
 就是把原来的conf.p 复制一份
 
### 配置文件信息修改

```
vim ../conf/me-2m-2s-async/broker.p

 
主机192.168.0.11(broker-a)修改后的配置文件信息如下

namesrvAddr=mqnameserver1:9876;mqnameserver2:9876;mqnameserver3:9876;mqnameserver4:9876

brokerIP1=192.168.0.11

brokerName=broker-a

brokerClusterName=TestCluster

brokerId=0

autoCreateTopicEnable=true

autoCreateSubscriptionGroup=true

rejectTransactionMessage=false

fetchNamesrvAddrByAddressServer=false

storePathRootDir=/root/store

storePathCommitLog=/root/store/commitlog

flushIntervalCommitLog=1000

flushCommitLogTimed=false

deleteWhen=04

fileReservedTime=72

maxTransferBytesOnMessageInMemory=262144

maxTransferCountOnMessageInMemory=32

maxTransferBytesOnMessageInDisk=65536

maxTransferCountOnMessageInDisk=8

accessMessageInMemoryMaxRatio=40

messageIndexEnable=true

messageIndexSafe=false

haMasterAddress=

brokerRole=ASYNC_MASTER

flushDiskType=ASYNC_FLUSH

cleanFileForciblyEnable=true

```

## 附加配置

```
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```
 

红色部分是根据模板文件修改的内容：

<font color="red"> `namesrvAddr=mqnameserver1:9876;mqnameserver2:9876;mqnameserver3:9876;mqnameserver4:9876`
</font>

指定broker注册的NameServer服务器
<font color="red"> 
`brokerIP1=192.168.0.11 (本机ip)`
</font>
强制指定本机IP，需要根据每台机器进行修改。官方介绍可为空，系统默认自动识别，但多网卡时IP地址可能读取错误

 <font color="red"> 
`brokerName=broker-a`
</font>

系统根据此名称确定主从关系，因此master和slave必须一致。每台机器的设定如下：

192.168.0.11 broker-a

192.168.0.14 broker-b

192.168.0.16 broker-a-s

192.168.0.17 broker-b-s

 <font color="red"> 
`brokerClusterName=TestCluster`
</font>

集群名称，可根据自己的需要修改。

 <font color="red"> 
`brokerId=0`
</font>

0 表示 Master， >0 表示 Slave，每台机器的设定如下：

192.168.0.11           `master     0`

192.168.0.14           `master     0`

192.168.0.16           `slave      1`

192.168.0.17           `slave      1`


 
 <font color="red"> 
`brokerRole=ASYNC_MASTER`
</font>

主从角色和是否异步，每台机器的设定如下：

192.168.0.11   `ASYNC_MASTER`

192.168.0.14   `ASYNC_MASTER`

192.168.0.16   `SLAVE`

192.168.0.17   `SLAVE`


参考:   http://sofar.blog.51cto.com/353572/1540874

---

# 然后依次部署就可以了