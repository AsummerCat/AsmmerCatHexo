---
title: NACOS集群和持久化配置
date: 2020-05-19 20:51:44
tags: [springcloud,nacos]
---

# NACOS集群和持久化配置

## 持久化配置

1. nacos-server/nacos/conf目录下找到sql脚本
2. 创建数据库 ->导入脚本
3. nacos-server/nacos/conf目录下找到application.propertis

```
打开配置文件中的 mysql相关配置 打开 并且配置mysql

spring.datasource.platform=mysql 
db.num=1
db.url=jdbc:mysql://127.0.0.1:3306/nacos_config
db.user=root
db.password=123456
```
<!--more-->

## 集群化配置

1. 指定端口启动 默认8848
```
启动命令后面加入 -p 3333
即为指定3333端口启动
比如 startup -p 3333

```

2. nacos集群配置cluster.conf 

在conf文件夹里有 修改下配置 
将集群节点配置进入其中 
```
比如:
192.168.1.1:3333
192.168.1.1:4444
192.168.1.1:5555
```
注意这个ip不是写127.0.0.1,
必须是hostname -i命令能识别的ip

3. 修改startup.sh 让其支持指定端口执行
```java
或者 直接修改application.proprtis的文件里的port端口

修改启动脚本:

while getopts ":m:f:s:" opt
do 
  case $opt in
    m)
```
![](/img/2020-05-19/nacos1.png)

![](/img/2020-05-19/nacos2.png)

修改后执行方式:
./start.sh -p 3333

4. 配置nginx 使其vip化 虚拟ip
```
配置反向代理
upsteam:xxxx
```