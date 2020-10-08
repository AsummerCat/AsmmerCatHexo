---
title: kafka笔记之安装部署及集群部署(二)
date: 2020-10-09 04:02:47
tags: [kafka笔记]
---

# 下载kafka
下载之后里面有内置的zk 可以用来启动

如果说 我们自己有安装zk,需要修改`config/server.properties`里面的`zookeeper.connect`属性   

## 启动命令
内置启动脚本
```
前台运行:
sh kafka-server-start.sh

还可以手动指定配置文件地址

sh kafka-server-start.sh ../config/server.properties

后台运行:
sh kafka-server-start.sh -daemon

```

## 临时日志文件地址
```
rm -rf /tmp/kafka-logs/
```
安装文档可以参考官网 

<!--more-->



# 搭建集群环境
## 第一步
需要修改`config/server.properties`里面的`zookeeper.connect`属性  
逗号分开

## 第二步
修改`config/server.properties`里面的`broker.id=xx`属性 ,这个要保证集群中唯一性 默认id都是0  

## 第三步
开启集群监听器

修改`config/server.properties`里面的`listeners=PLAINTEXT://192.168.1.1:9092`
修改成集群中 当前机子的ip地址

这样就已经简单的配置了集群环境

 