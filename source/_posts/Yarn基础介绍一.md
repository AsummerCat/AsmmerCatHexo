---
title: Yarn基础介绍一
date: 2022-08-01 16:12:05
tags: [大数据,hadoop,yarn]
---
# Yarn基础介绍-资源调度

Yarn是一个资源调度平台,负责为运算程序提供服务器运算资源
```
主要负责:
1.集群管理
2.给任务合理分配资源
```

## 基础架构
Yarn主要由`ResourceManager`,`NodeManager`,`ApplicationMaster`,`Container`等组件构成;

`ResourceManager`全局监控管理,
`NodeManager`单个节点的处理,
`Container`单个节点下分配的资源,
`ApplicationMaster`分配资源下的任务管理者

1.ResourceManager(RM)
```
1.处理客户端请求
2.监控NodeManager
3.启动或监控ApplicaitionMaster
4.资源的分配和调度
```
<!--more-->
2.NodeManager(NM)
```
1.管理单个节点上的资源
2.处理来自ResourceManager的命令
3.处理来自ApplicaitionMaster的命令
```

3. ApplicationMaster(AM)
```
1.为应用程序申请并分配给内部的任务
2.任务的监控和容错
```

4.Container
```
是Yarn中资源抽象,它封装了某个节点上的多维度资源,
如内存,CPU,磁盘,网络等
```