---
title: gateway网关整合eureka
date: 2019-10-08 15:06:11
tags: [gateway,springCloud,eureka,微服务]
---

# [demo地址](https://github.com/AsummerCat/gateway-demo)

# 项目由 gradle构建



# 整合注册中心

## 首先还是注册服务开启 包导入清楚后开始配置

(略)

<!--more-->

## gateway网关配置

```java
spring:
  application:
    name: gateway-rureka

  cloud:
    gateway:
      ## 默认根据服务自动匹配
      discovery:
        locator:
          enabled: true
          ## 服务名称小写
          lowerCaseServiceId: true

          # 自定义路由
      routes:
        - id: web-server
          # lb: 后面是服务名称
          uri: lb://WEB-SERVER
          predicates:
            - Path=/web/**
          filters:
            - StripPrefix=1

server:
  port: 8091

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8100/eureka

```

uri以lb://开头（lb代表从注册中心获取服务），后面接的就是你需要转发到的服务名称，这个服务名称必须跟eureka中的对应，否则会找不到服务,错误如下:

```java
org.springframework.cloud.gateway.support.NotFoundException: Unable to find instance for fsh-house1
```



##  启用禁用配置

关于路由规则什么的我们后面再做介绍，本章只是先体验下Spring Cloud Gateway的功能，能够创建一个新的项目，成功启动就可以了，一步步来。

如果你的项目中包含了spring-cloud-starter-gateway，但你不想启动网关的时候可以通过下面的配置禁用掉：

**application.properties**

```
spring.cloud.gateway.enabled=false.
```

如果引入了spring-cloud-starter-netflix-eureka-client包，但你不想整合Eureka，也可以通过下面的配置关闭：

```java
eureka.client.enabled=false
```

# 默认自动配置

说完了直接配置路由的方式，我们来说说不配置的方式也能转发，有用过Zuul的同学肯定都知道，Zuul默认会为所有服务都进行转发操作，只需要在访问路径上指定要访问的服务即可，通过这种方式就不用为每个服务都去配置转发规则，当新加了服务的时候，不用去配置路由规则和重启网关。

在Spring Cloud Gateway中当然也有这样的功能，只需要通过配置即可开启，配置如下：

```
spring.cloud.gateway.discovery.locator.enabled=true
```

开启之后我们就可以通过地址去访问服务了，格式如下：

```java
http://网关地址/服务名称（大写）/**

http://localhost:8084/FSH-HOUSE/house/1
```

这个大写的名称还是有很大的影响，如果我们从Zull升级到Spring Cloud Gateway的话意味着请求地址有改变，或者重新配置每个服务的路由地址，通过源码我发现可以做到兼容处理，再增加一个配置即可：

```
spring.cloud.gateway.discovery.locator.lowerCaseServiceId=true
```

配置完成之后我们就可以通过小写的服务名称进行访问了，如下：

```java
http://网关地址/服务名称（小写）/**

http://localhost:8084/fsh-house/house/1
```

