---
title: TIDB基本概念
date: 2022-08-01 13:56:42
tags: [TIDB]
---

TIDB基本概念
# 三大核心组件
TIDB Server, PD Server , TIKV Server  还有其他TI Spark用来离线计算
## TIDB Server
```
负责接收SQL请求的(解析sql),通过PD中存储的元数据找到数据存在哪里个TIKV上,
并于TIKV进行交互,将结果返回给用户;
```
<!--more-->
## PD Server
```
整个集群管理者  (主要存储一些元数据) 如有哪些表;

可以进行
负载均衡,
分配全局唯一的事务ID;
```

## TIKV Server
```
负责 真正存储数据的;
本质上是一个KV存储引擎;
```

## TI Spark
```
它是用来解决永久的复杂的OLAP的查询需求的
(分析数据)
```

## TIDB Operator
```
用来方便云上部署的组件
```

## 数据迁移工具 TIDB Lightning
![](/img/2022-08-01/1.png)
![](/img/2022-08-01/2.png)
![](/img/2022-08-01/3.png)

## TIBD架构
![](/img/2022-08-01/4.png)

## 标准业务处理流程
![](/img/2022-08-01/5.png)

## 部署方式
![](/img/2022-08-01/6.png)


