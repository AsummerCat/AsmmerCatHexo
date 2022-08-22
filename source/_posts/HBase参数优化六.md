---
title: HBase参数优化六
date: 2022-08-22 11:33:29
tags: [大数据,HBase]
---
# HBase参数优化


### 1.Zookeeper 会话超时时间
hbase-site.xml
```
属性：zookeeper.session.timeout
解释：默认值为 90000 毫秒（90s）。当某个 RegionServer 挂掉，90s 之后 Master 才
能察觉到。可适当减小此值，尽可能快地检测 regionserver 故障，可调整至 20-30s。
看你能有都能忍耐超时，同时可以调整重试时间和重试次数
hbase.client.pause（默认值 100ms）
hbase.client.retries.number（默认 15 次）
```
<!--more-->
### 2.设置 RPC 监听数量
hbase-site.xml
```
属性：hbase.regionserver.handler.count
解释：默认值为 30，用于指定 RPC 监听的数量，可以根据客户端的请求数进行调整，读写
请求较多时，增加此值。
```
<!--more-->
### 3.手动控制 Major Compaction
hbase-site.xml
```
属性：hbase.hregion.majorcompaction
解释：默认值：604800000 秒（7 天）， Major Compaction 的周期，若关闭自动 Major
Compaction，可将其设为 0。如果关闭一定记得自己手动合并，因为大合并非常有意义
```

### 4.优化 HStore 文件大小
hbase-site.xml
```
属性：hbase.hregion.max.filesize
解释：默认值 10737418240（10GB），如果需要运行 HBase 的 MR 任务，可以减小此值，
因为一个 region 对应一个 map 任务，如果单个 region 过大，会导致 map 任务执行时间过长。该值的意思就是，
如果 HFile 的大小达到这个数值，则这个 region 会被切分为两
个 Hfile。
```

### 5.优化 HBase 客户端缓存
hbase-site.xml
```
属性：hbase.client.write.buffer
解释：默认值 2097152bytes（2M）用于指定 HBase 客户端缓存，增大该值可以减少 RPC
调用次数，但是会消耗更多内存，反之则反之。一般我们需要设定一定的缓存大小，以达到
减少 RPC 次数的目的。
```

### 6.指定 scan.next 扫描 HBase 所获取的行数
hbase-site.xml
```
属性：hbase.client.scanner.caching
解释：用于指定 scan.next 方法获取的默认行数，值越大，消耗内存越大。
```

### 7.BlockCache 占用 RegionServer 堆内存的比例
hbase-site.xml
```
属性：hfile.block.cache.size
解释：默认 0.4，读请求比较多的情况下，可适当调大
```

### 8.MemStore 占用 RegionServer 堆内存的比例
hbase-site.xml
```
属性：hbase.regionserver.global.memstore.size
解释：默认 0.4，写请求较多的情况下，可适当调大
```

# JVM调优
JVM 调优的思路有两部分：一是内存设置，二是垃圾回收器设置。

### 1.设置使用 CMS 收集器
```
-XX:+UseConcMarkSweepGC
```

### 2.保持新生代尽量小，同时尽早开启 GC，例如：
```
//在内存占用到 70%的时候开启 GC -XX:CMSInitiatingOccupancyFraction=70
//指定使用 70%，不让 JVM 动态调整
-XX:+UseCMSInitiatingOccupancyOnly
//新生代内存设置为 512m
-Xmn512m
//并行执行新生代垃圾回收
-XX:+UseParNewGC
// 设 置 scanner 扫 描 结 果 占 用 内 存 大 小 ， 在 hbase-site.xml 中，设置
hbase.client.scanner.max.result.size(默认值为 2M)为 eden 空间的 1/8
（大概在 64M）
// 设置多个与 max.result.size * handler.count 相乘的结果小于 Survivor 
Space(新生代经过垃圾回收之后存活的对象)
```

# HBase 使用经验法则
![image](/img/2022-08-22/16.png)