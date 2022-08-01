---
title: Yarn的调度器三
date: 2022-08-01 16:13:43
tags: [大数据,hadoop,yarn]
---
# Yarn的调度器

这边主要就是 向`ResourceManager`提交了很多任务,
生成一个队列,依次消费,这里就是调度器(调度队列)
```
主要又三种

1. 先进先出调度器 (FIFO)

2. 容量调度器(Capacity Scheduler)

3. 公平调度器( Fair Scheduler)
```
<!--more-->

## 具体配置详见: `yarn-default.xml`
Hadoop默认调度器是: `CapacityScheduler` ->容器调度器

CDH默认调度器是: `Fair Scheduler` ->公平调度器

以下参数可修改调度器(value)
```
<property>
  <description>The class to use as the resource scheduler.</description>
  <name>yarn.resourcemanager.scheduler.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
<property>
```


## 先进先出调度器 (FIFO) ,基本不用

单队列,根据提交作业(job)的先后顺序,先来先服务;


## 容量调度器(Capacity Scheduler)
Yahoo开发的多用户调度器
```
1.多队列 ,每个队列可配置一定的资源量,每个对垒采取FIFO(先进先出)调度策略

2.容量保证: 管理员可为每个队列设置资源最低保证和资源使用上限

3.灵活性: 如果一个队列中的资源有剩余,可以暂时共享给那些需要资源的队列,
而一旦该队列有新的应用程序提交,这其他队列借调的资源会归还给该队列

4.多租户:
  支持多用户共享集群和应用程序同时运行.
  为了防止同一个用户的作业独占队列中的资源,
  该调度器会对同一用户提交的作业所占资源量进行限定;
```


## 公平调度器( Fair Scheduler)
Facebook 开发的多用户调度器
<font color='red'>特点:</font>
```
同队列所有任务共享资源,在时间尺度上获得公平的资源
```

跟容量调度器大体相同;
```
1.多队列

2.容量保证

3.灵活性

4.多租户
```

`不同:`
1.核心调度策略不同

| 调度器 |特点  |
| --- | --- |
| 容量调度器 |优先选择<font color='red'>资源利用率低</font>的队列 (默认) |
| 公平调度器 | 优先选择对资源的<font color='red'>缺额</font>比例大的队列  (又名:新增资源->平均分配) (数据量巨大的情况,考虑)|

2. 每个队列可以单独设置资源分配方式不同

| 调度器 |资源分配方式 |
| --- | --- |
| 容量调度器 |FIFO <font color='red'>DRF</font> |
| 公平调度器 | FIFO <font color='red'>FAIR  DRF</font>  |


