---
title: Yarn生产环境核心配置参数五
date: 2022-08-01 16:14:54
tags: [大数据,hadoop,yarn]
---
# Yarn生产环境核心配置参数
配置存放位置: `yarn-site.xml`
## 1.ResourceManager相关
```
1. yarn.resourcemanager.scheduler.class  
配置调度器,默认容量调度器

2. yarn.resourcemanager.scheduler.client.thread-count 
ResourceManager处理调度器请求的线程数量,默认50(表示客户端连接RM最大的连接数)
(根据总集群的CPU进程数修改)
```
<!--more-->

## 2.NodeManager相关
```
1. yarn.nodemanager.resource.detect-hardware-capabilities
是否让yarn自己检验硬件配置进行配置,默认false


2. yarn.nodemanager.resource.count-logical-processors-as-cores
是否将虚拟核心数当做CPU核数,默认false


3. yarn.nodemanager.resource.pcores-vcorses-multiplier
虚拟核心数和物理核心数乘数 ,例如
4核8线程,该参数就应设2 ,默认1.0


4. yarn.nodemanager.resource.memory-mb
NodeManager使用内存 ,默认8G

5. yarn.nodemanager.resource.system-reserved-memory-mb
NodeManager为系统保留多少内存,以上两个参数配置一个即可

6. yarn.nodemanager.resource.cpu-vcores
NodeManager使用CPU核心数,默认8个


7. yarn.nodemanager.pmem-check-enabled  (监控机制)
是否开启物理内存检测限制container,默认打开

8. yarn.nodemanager.vmem-check-enabled (监控机制 默认true)
是否开启虚拟内存检测限制container,默认打开,
推荐关闭false,
因为linux系统会为java进程预留虚拟内存,
但是java1.8不去使用这部分内容,
如果超出分配值,则会被kill掉

9. arn.nodemanager.vmem-pmem-ratio (虚拟内存转换物理内存)
虚拟内存物理内存比例, 默认2.1


```

## Container 相关
```
1. yarn.scheduler.minimum-allocation-mb
容器最小内存,默认1G


2. yarn.scheduler.maximum-allocation-mb
容器最大内存,默认8G

3. yarn.scheduler.minimum-allocation-vcores
容器最小CPU核心数,默认1个

4. yarn.scheduler.maximum-allocation-vcores
容器最大CPU核心数,默认4个

```

## queue队列相关
# 队列中任务的优先级
容器调度器,支持任务优先级的配置,在好资源紧张时候,优先级高的任务将优先获取资源;

默认情况,Yarn将所有任务的优先级限制为0,若先使用任务优先级功能,需要开饭改限制;

### 1.修改`yarn-site.xml`文件,增加以下参数
`0-5` 数值越大优先级越高
```
<property>
   <name>yarn.cluster.max-application-priority</name>
   <value>5</value>
<property>
```

### 2.分发配置,重启yarn
```
sbin/stop-yarn.sh
sbin/start-yarn.sh
```

### 如何调整任务优先级
1.命令行运行
```
运行jar的时候指定优先级
-D mapreduce.job.priority=5

hadoop jar xxx.jar 启动类全类名 -D mapreduce.job.priority=5e /input /output
```
1.1 修改正在运行的任务的优先级
```
yarn application -appID 任务的applicationId -updatePriority 5
```

2.job运行
```
//指定优先级
conf.set("mapreduce.job.priority","5");
``` 