---
title: 避免SpringBoot使用actuator漏洞
date: 2023-01-11 11:12:42
tags: [SpringBoot,actuator]
---
## 指定开启actuator需要的模块
https://start.aliyun.com


## demo地址
```
https://github.com/AsummerCat/SpringActuatorDemo.git
```
### 请求路径
```
注意: 2.x之后的sptingboot版本 路径需要加/actuator/xxx

http://localhost:8081/actuator/info
```
###  导入依赖
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

<!--more-->

### 配置文件开启
```
server:
  port: 8081

spring:
  application:
    name: SpringActuatorDemo

management:
  endpoints:
    web:
      exposure:
        #        include: [info,health,beans,env,configprops,beans,dump,mappings,metrics,trace]

        # info 节点        自定义配置信息 (必开)
        # health节点       心跳监控节点   (必开)
        # beans节点        bean装配信息可以在这里看到
        # env节点          获取全部(系统)环境变量属性
        # configprops节点  描述配置属性（包含默认值）如何注入Bean 程序的默认配置属性
        # mappings节点     描述全部的URI路径，以及它们和控制器（包含Actuator端点）的映射关系
        # metrics节点      报告各种应用程序度量信息，比如内存用量和HTTP请求计数 比如线程啥的 显示在Details上
        #trace节点         提供基本的HTTP请求跟踪信息（时间戳、HTTP头等）
        #shutdown节点      关闭应用程序，要求 endpoints.shutdown.enabled 设置为 true
        #heapdump         返回一个GZip压缩的JVM堆dump
        #autoconfig       提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过
        #dump             获取线程活动的快照

        #开启所有节点 不设置的话 默认开启 info 和 health端点
#        include: "*"
       include: [info,health,heapdump]
        #忽略节点
      #  exclude: [env,beans]
  #具体开启明细 可以看到redis db啥的
  endpoint:
    health:
      show-details: always

```