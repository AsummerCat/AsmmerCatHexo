---
title: docker容器间相互通信
date: 2020-06-12 14:43:00
tags: [docker]
---

# docker容器间相互通信


## 单向通行
进行网络配置后 直接可以用容器名称 进行通信

这样就可以直接用容器单向通信了
在jdbc中直接用容器名称 直接连接

### 创建容器时 添加额外参数
--link 需要连接的容器名称 
```
docker run -d   镜像  --name tomcat 
--link database
```
<!--more-->
### 测试
```
在tomcat中
ping database 可以访问

```

## Bridge网桥双向通信

多个容器在同一个网桥内默认通信  
(默认情况下都在默认网桥下)
让多个容器双向通信

查看网桥: `docker network |s`  
新建网桥: `docker network create -d bridge 自定义网桥名称`

容器绑定网桥:`docker network connect 网桥名称 容器名称`

```
这样直接连接 同网桥下的容器 直接容器名称 就可以通信了
```

# 共享容器挂载点
实现数据共享

```
创建共享容器 
docker create --name webpage -v /webapps :/tomcat/webapps tomcat /bin/true
(该容器不运行)

共享容器挂载点
docker run --vilume-from webpage --name t1 -d tomcat


```