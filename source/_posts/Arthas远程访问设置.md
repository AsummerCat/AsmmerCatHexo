---
title: Arthas远程访问设置
date: 2021-03-03 17:48:12
tags: Arthas
---

# [arthas下载地址](https://github.com/alibaba/arthas/releases)
# 方式一: 
直接启动`java -jar arthas-boot.jar`

开启远程连接启动:
```
--target-ip: 要访问的ip
注意需要在被监控端上开启arthas  并且不安全

java -jar /home/arthas-boot.jar --target-ip 27.154.73.55
```
<!--more-->

# 方式二(推荐):
tunnel server方式远程
1.
```
arthas-tunnel-server-3.4.8-fatjar.jar

使用arthas tunnel server连接远程arthas
启动远程连接服务：端口默认为8080

java -jar  -Dserver.port=8081  /home/arthas-tunnel-server-3.4.8-fatjar.jar>/home/arthas-tunnel-server.log &
```
2.启动arthas
```
是否后台启动加入前缀: nohup

java -jar /home/arthas-boot.jar --tunnel-server 'ws://112.74.43.136.7777/ws' --agent-id mytest123456


```
```
agent-id这个是用来连接的密码
```
3. 
注意上面的接口中的红线框部分是agentId。后面连接会用到。

http://192.168.232.100:8081/   

192.168.232.111：是我启动服务是配合的ip地址也就是服务器的地址。端口8081是我自定义的。


# 方式三(推荐,整合springboot):

嵌入Springboot项目中

1. 启动tunnel server
```
arthas-tunnel-server-3.4.8-fatjar.jar

java -jar  -Dserver.port=8081  /home/arthas-tunnel-server-3.4.8-fatjar.jar>/home/arthas-tunnel-server.log &
```
2.导入pom.xml
```
        arthas-spring-boot-starter

```
3.配置文件
```
arthas.agent-id=hsehdfsfghhwertyfad
arthas.tunnel-server=ws://47.75.156.201:7777/ws
```
4.启动顺序
先启动tunnel server

- ->再启动项目注册到tunnelServer