---
title: springboot整合eureka和springbootadmin进行监控服务健康状态
date: 2019-08-01 22:44:41
tags: [actuator,SpringCloud,SpringBoot,springBootAdmin]
---

# 利用SpringBootAdmin 进行服务健康监控的可视化

## [demo地址](https://github.com/AsummerCat/spring-boot-admin-demo)

## 首先老规矩导入pom文件

### 监控端

```
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
      
   <dependency>
      <groupId>de.codecentric</groupId>
      <artifactId>spring-boot-admin-starter-server</artifactId>
   </dependency>
		
```



### 被监控端 —>客户端

```
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
      
   <dependency>
      <groupId>de.codecentric</groupId>
      <artifactId>spring-boot-admin-starter-client</artifactId>
   </dependency>
```

<!--more-->

# 监控端实现

## 启动类添加

```
@EnableAdminServer
```



# 客户端实现

## 配置文件

```
spring:
  application:
    name: admin-client

    #注册到监控端
  boot:
    admin:
      client:
        url: http://localhost:9092
        user: 如果有账号的话 添加这两句
        password: 如果有密码控制的话
```

## 暴露你需要暴露的端点

这边具体端点 在另外一篇端口介绍里有写

```
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

      #开启所有节点 不设置的话 默认开启 info 和 health端点
        include: "*"
        #忽略节点
      #  exclude: [env,beans]
  #具体开启明细 可以看到redis db啥的
  endpoint:
    health:
      show-details: always


# info节点会显示一下 info 开头的内容
info:
  test: 001
  test1: 002
```



这样简单的就实现了

# 整合eureka实现

这边会自动监控微服务内的健康状态

只要把 监控端 和服务端全都注册到eureka就可以了 

