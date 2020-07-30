---
title: K8S笔记Service相关(五)
date: 2020-07-30 09:28:48
tags: [K8S笔记]
---

# K8S笔记Service相关(五)
## Server定义
Service是Kubernetes最核心概念，通过创建Service,  
可以为一组具有相同功能的容器应用提供一个统一的入口地
址，  
并且将请求负载分发到后端的各个容器应用上。  

## Service的解析

yaml格式的Service定义文件

<!--more-->

```
apiVersion: v1
kind: Service
matadata:
  name: string
  namespace: string
  labels:
  - name: string
  annotations:
  - name: string
spec:
  selector: []
  type: string
  clusterIP: string
  sessionAffinity: string
  ports:
  - name: string
    protocol: string
    port: int
    targetPort: int
    nodePort: int
status:
    loadBalancer:
      ingress:
        ip: string
        hostname: string

```

## Service的参数定义
![Service的参数定义1](/img/2020-07-25/19.png)  
![Service的参数定义2](/img/2020-07-25/20.png)  
![Service的参数定义3](/img/2020-07-25/21.png)  

## Server的基本用法
一般来说，对外提供服务的应用程序需要通过某种机制来实现，  
对于容器应用最简便的方式就是通过TCP/IP机制及
监听IP和端口号来实现。  
创建一个基本功能的Service  

![Server的基本用法](/img/2020-07-25/22.png)  
![Server的基本用法](/img/2020-07-25/23.png)  
#### 多端口Service 
有时一个容器应用也可能需要提供多个端口的服务，那么在Service的定义中也可以相应地设置为将多个端口对应 到多个应用服务。
 ![多端口Service](/img/2020-07-25/24.png) 

#### 外部服务Service
在某些特殊环境中，应用系统需要将一个外部数据库作为后端服务进行连接，或将另一个集群或Namespace中的
服务作为服务的后端，这时可以通过创建一个无Label Selector的Service来实现
 ![外部服务Service](/img/2020-07-25/25.png) 