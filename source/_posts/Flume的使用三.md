---
title: Flume的使用三
date: 2022-08-12 09:41:31
tags: [Flume,大数据]
---
# Flume的使用

## 数据源机子上编写配置文件`flume.conf`

<!--more-->

```
# example.conf：单节点 Flume 配置

# 命名此代理上的组件
a1.sources  =  r1 
a1.sinks  =  k1 
a1.channels  =  c1

# 描述/配置源方式一:  客户端的端口和调用方式
a1.sources.r1.type  =  netcat 
a1.sources.r1.bind  =  localhost 
a1.sources.r1.port  =  44444
# 描述/配置源方式二: 监控打印日志
a1.sources.r1.type  =  exec 
a1.sources.r1.command  =  tail -F /xxx/xxx/logs/hive.log


# 描述接收器 方式一: 记录INFO级别的日志，一般用于调试
a1.sinks.k1.type  =  logger

# 描述接收器 方式二: HDFS Sink 此Sink将事件写入到Hadoop分布式文件系统HDFS中
a1.sinks.k1.type =hdfs
a1.sinks.k1.hdfs.path=hdfs://0.0.0.0:9000/flume/%Y%m%d
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix= logs-
#是否按照实际滚动文件夹
a1.sinks.k1.hdfs.round = true
# 多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
# 重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
# 是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型,可支持压缩 (SequenceFile , DataStream , CompressedStream)
a1.sinks.k1.hdfs.fileType = DataStream
# 多久生成一个新的文件 (1小时)
a1.sinks.k1.hdfs.rollInterval = 3600
#设置每个文件的滚动大小 (接近128M)
a1.sinks.k1.hdfs.rollSize = 134217700
# 文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0



# 描述接收器 方式三: File Roll Sink 在本地文件系统中存储事件
a1.sinks.k1.type   = file_roll
a1.sinks.k1.sink.directory =  /home/park/work/apache-flume-1 .6.0-bin /mysink


# 使用在内存中缓冲事件的通道
a1.channels.c1.type  =  memory 
a1.channels.c1.capacity  =  1000 
a1.channels.c1.transactionCapacity  =  100

# 将source和sink绑定到channel 
a1.sources.r1.channels  =  c1 
a1.sinks.k1.channel  =  c1
```

### 启动
```
bin/flume-ng agent --conf conf --conf-file example.conf --name a1
```
或者
```
bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties.template
```

### 多配置文件启动
```
 bin/flume-ng agent --conf conf --conf-file example.conf --conf-uri http://localhost:80/flume.conf --conf-uri http://localhost:80/override.conf --name a1
```

### 开启配置日志记录和原始数据日志记录
```
bin/flume-ng agent --conf conf --conf-file example.conf --name a1
-Dorg.apache.flume.log.printconfig=true -Dorg.apache.flume.log.rawdata=true

开启日志级别
-Dflume.root.logger=INFO console
```

## 实时监控单个追加文件
```
# 命名此代理上的组件
a1.sources  =  r1 
a1.sinks  =  k1 
a1.channels  =  c1

# 描述/配置源方式: 监控打印日志
a1.sources.r1.type  =  exec 
a1.sources.r1.command  =  tail -F /xxx/xxx/logs/hive.log


# 描述接收器 方式一: 记录INFO级别的日志，一般用于调试
a1.sinks.k1.type  =  logger



# 使用在内存中缓冲事件的通道
a1.channels.c1.type  =  memory 
a1.channels.c1.capacity  =  1000 
a1.channels.c1.transactionCapacity  =  100

# 将source和sink绑定到channel 
a1.sources.r1.channels  =  c1 
a1.sinks.k1.channel  =  c1
```


## 实时监控目录下的多个新文件
上传完毕之后,会在文件结尾标记`.COMPLETED`;

案例: 监控文件夹读取新文件,推送HDFS
```
# 命名此代理上的组件
a1.sources  =  r1 
a1.sinks  =  k1 
a1.channels  =  c1

# 描述/配置源方式: 监控文件夹
a1.sources.r1.type  =  spooldir 
a1.sources.r1.spoolDir  =  /xxx/upload 
a1.sources.r1.fileSuffix = .COMPLETED
a1.sources.r1.fileHeader = true
#忽略所有以.tmp结尾的文件,不上传
a1.sources.r1.ignorePattern = ([^ ]*\.tmp)

# 描述接收器 方式二: HDFS Sink 此Sink将事件写入到Hadoop分布式文件系统HDFS中
a1.sinks.k1.type =hdfs
a1.sinks.k1.hdfs.path=hdfs://0.0.0.0:9000/flume/upload/%Y%m%d
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix= upload-
#是否按照实际滚动文件夹
a1.sinks.k1.hdfs.round = true
# 多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
# 重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
# 是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型,可支持压缩 (SequenceFile , DataStream , CompressedStream)
a1.sinks.k1.hdfs.fileType = DataStream
# 多久生成一个新的文件 (1小时)
a1.sinks.k1.hdfs.rollInterval = 3600
#设置每个文件的滚动大小 (接近128M)
a1.sinks.k1.hdfs.rollSize = 134217700
# 文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0

# 使用在内存中缓冲事件的通道
a1.channels.c1.type  =  memory 
a1.channels.c1.capacity  =  1000 
a1.channels.c1.transactionCapacity  =  100

# 将source和sink绑定到channel 
a1.sources.r1.channels  =  c1 
a1.sinks.k1.channel  =  c1
```


## 实时监控多目录下的多个追加文件
<font color='red'>Taildir Source</font>
适用于监听多个实时追加文件,并且能够实现断点续传
```
# 命名此代理上的组件
a1.sources  =  r1 
a1.sinks  =  k1 
a1.channels  =  c1

# 描述/配置源方式: 监控文件夹实时追加文件
a1.sources.r1.type  =  TAILDIR 
#断点续传记录文件夹 默认路径(~/.flume/taildir_position.json)
a1.sources.r1.positionFile = /tail_dir.json
a1.sources.r1.filegroups = f1 f2
# 正则表达式 读取的文件名称
a1.sources.r1.filegroups.f1 = /log/.*file.*
a1.sources.r1.filegroups.f2 = /file2/log/.*log.*


# 描述接收器 方式二: HDFS Sink 此Sink将事件写入到Hadoop分布式文件系统HDFS中
a1.sinks.k1.type =hdfs
a1.sinks.k1.hdfs.path=hdfs://0.0.0.0:9000/flume/upload/%Y%m%d
#上传文件的前缀
a1.sinks.k1.hdfs.filePrefix= upload-
#是否按照实际滚动文件夹
a1.sinks.k1.hdfs.round = true
# 多少时间单位创建一个新的文件夹
a1.sinks.k1.hdfs.roundValue = 1
# 重新定义时间单位
a1.sinks.k1.hdfs.roundUnit = hour
# 是否使用本地时间戳
a1.sinks.k1.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a1.sinks.k1.hdfs.batchSize = 100
#设置文件类型,可支持压缩 (SequenceFile , DataStream , CompressedStream)
a1.sinks.k1.hdfs.fileType = DataStream
# 多久生成一个新的文件 (1小时)
a1.sinks.k1.hdfs.rollInterval = 3600
#设置每个文件的滚动大小 (接近128M)
a1.sinks.k1.hdfs.rollSize = 134217700
# 文件的滚动与Event数量无关
a1.sinks.k1.hdfs.rollCount = 0

# 使用在内存中缓冲事件的通道
a1.channels.c1.type  =  memory 
a1.channels.c1.capacity  =  1000 
a1.channels.c1.transactionCapacity  =  100

# 将source和sink绑定到channel 
a1.sources.r1.channels  =  c1
a1.sinks.k1.channel  =  c1
```
可能存在不同log框架 会出问题
比如:log4j 次日生成的文件会将源文件备份成log.日期
导致抓取文件失败
logback则是每日新建的时候就会带上时间戳,所以不会有问题

解决方案: 修改源码
原flume断点续传是根据->node+文件名
修改为仅node
```
ReliableTaildirEventReader,java

查找 注释掉->
if(tf ==null||!tf.getPath().equals(f.getAbsolutePath()))
```