---
title: eureka高可用的实现互相注册
date: 2019-01-10 15:50:55
tags: [SpringCloud,eureka]
---

# eureka实现高可用 

避免如果当机一台注册中心 服务全部不可用



<!--more-->

# 修改host 当做三台不同的测试机

```java
# eureka测试环境
127.0.0.1 localhost
127.0.0.1 server1
127.0.0.1 server2
```



#  euerka互相注册实现高可用



```java
spring:
  application:
    name: eureka-Server
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    serviceUrl:
      defaultZone: http://server1:8100/eureka,http://server2:8100/eureka

  server:
    enable-self-preservation: false
  instance:
    hostname: localhost

server:
  port: 8080

management:
  security:
    enabled: false
```

其他两台配置相同 只要修改defaultZone 这里注册的给其他两台

hostname 这里也修改成不同的

主要注意的是要开启  register-with-eureka: true ,fetch-registry: true  这样才能被注册到其他的

![高可用](/img/2019-1-10/eureka高可用.png)

![高可用](/img/2019-1-10/eureka高可用2.png)

这里只注册了两台

**ps:如果注册报错不用管 启动三条之后就不会报错了 因为服务没启动注册不上**

----



# 服务如何注册到eureka 

```java
spring:
  application:
    name: test-server
eureka:
  client:
    fetch-registry: true
    serviceUrl:
      defaultZone: http://localhost:8080/eureka,http://server1:8100/eureka
server:
  port: 8904

management:
  security:
    enabled: false
```

修改defaultZone属性 以逗号隔开

这样就成功注册到eureka了 

会同时注册到 localhost 和server1 两个eureka实现高可用



# 还有一种高可用方式 使用域名实现负载均衡

# 完成

这样就成功注册到eureka了