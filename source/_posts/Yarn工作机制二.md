---
title: Yarn工作机制二
date: 2022-08-01 16:13:10
tags: [大数据,hadoop,yarn]
---
# Yarn工作机制

## 流程

1.MapReduct程序->MR程序提交到客户端所在的节点

2.`YarnRunner`向`ResourceManager`申请一个`Application`

3.`RM`将该应用程序的资源路径返回给`YarnRunner`
```
返回: 
  1.Application资源提交路径  hdfs://xxx/.staging/application_id
  2.application_id
```
<!--more-->
4. 提交job运行所需要的资源到资源路径上
```
hdfs://xxx/.staging/application_id

job.split  切片信息
job.xml  运行参数
wc.jar 程序代码

这些文件在job.sumbmit()后生成
```

5.资源提交完毕后,申请运行myAppMaster

6.`ResourceManager`内部将用户请求初始化为一个Task
这里会用到调度器:
```
如果有很多Task,会放到调度队列中,一个个读取
```

7.空闲的`NodeManager`会从队列中领取Task任务

8.领取任务后,`NodeManager`会创建容器`Container`(里面有CPU+RAM)

9. 在容器`Container`中启动一个进程`ApplicationMaster`

10. 当前执行的进程`ApplicationMaster`会去下载job资源到本地 根据切片信息再去申请容器跑Task任务
```
job.split  切片信息
```
11.`ApplicationMaster`拿到切片信息之后,申请运行对应数量的`MapTask`容器

12.空闲的`NodeManager`再去领取Task任务,创建容器,拷贝文件(cpu+ram+jar)

13.`ApplicationMaster`会发送程序启动运行脚本给Task对应的容器,开启`yarnClild`(子进程)运行,数据处理完毕之后,处理结果持久化磁盘,然后
返回通知给`ApplicationMaster`

14.`ApplicationMaster`获取通知之后,再去向`ResourceManager`申请运行`ReduceTask`任务

15.仍然是 `NodeManager`领取Task任务,创建容器

16. `ApplicationMaster`发送命令通知`Task`的容器处理

17. `ReduceTask`向`MapTask`获取相应分区的数据,进行处理

18.处理完毕之后,通知 `ApplicationMaster`,
然后`ApplicationMaster`,
再对`ResourceManager`申请运行完毕要释放资源,包括释放`ApplicationMaster`


