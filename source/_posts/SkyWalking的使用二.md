---
title: SkyWalking的使用二
date: 2022-02-09 22:18:35
tags: [链路跟踪,SkyWalking]
---

# SkyWalking的使用

# 两种方式


## 使用war包发布
```
需要修改tomcat的相关配置

1. vi tomcat/bin/cataina.sh 
在顶部新增几行配置
CATALINA_OPTS="$CATALINA_OPTS -javagent: SkyWalking的安装目录/agent/skywalking-agent.jar"; export CATALINA_OPTS

ps: 注意这是在一行上的内容

```
<!--more-->

## 使用springboot发布
```
1.修改SkyWalking的配置文件
cd agent/config

vi agent.config

修改:
agent.service_name={SW_AGENT_NAME:your_applicationame}

为你的项目名称:
agent.service_name={SW_AGENT_NAME:test1}


2.使用的探针agent是 ->skywalking_springboot.jar

3.springboot启动命令中添加探针 
因为默认不会集成到springboot中

命令:
java -javagent: SkyWalking的安装目录/agent/skywalking_springboot.jar -jar xxx.jar&"

```

### 使用探针(探针配置)命令详解
```
java -javagent: SkyWalking的安装目录/agent/skywalking-agent.jar 
-Dskywalking.agent.service_name=a-service
-Dskywalking.collector.backend_service=192.168.31.10.11800
-Dskywalking.logging.file_name=a-service-api.log
-jar xxx.jar&"

详情:
-Dskywalking.agent.service_name 指定当前服务名称
-Dskywalking.collector.backend_service 指定上报的服务器
-Dskywalking.logging.file_name 指定上报后存储在skywalking上的日志名称

```
