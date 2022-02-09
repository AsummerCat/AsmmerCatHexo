---
title: SkyWalking集成springboot实现java代码获取TraceId四
date: 2022-02-09 22:20:47
tags: [链路跟踪,SkyWalking]
---

# SkyWalking集成springboot实现java代码获取TraceId

这样的目的是更好的在代码中最终日志的时候获取链路ID 进行定位问题


## SkyWalking提供了追踪Trace的工具包
pom中添加
```
<dependency>
  <groupId>org.apache.skywalking</groupId>
  <artifactId>apm-toolkit-trace</artifactId>
  <version>${skywalking.version}</version>
</dependency>


${skywalking.version} 这个跟 SkyWalking版本号一致
```
<!--more-->

## 运行springboot

```
java -javaagent:/path/xxx-agent.jar=agent.service_name=项目名称 -jar xxx.jar&
```



## 使用
```
在代码中使用

获取traceId: 
TraceContext.traceId(); 

使当前链路报错,并且提示报错信息:
ActiveSpan.error(new RuntimeException("Test-Error-Throwable"));

打印Info信息:
ActiveSpan.info("INFO-MSG");

打印Debug信息:
ActiveSpan.debug("DEBUG-MSG");

```
