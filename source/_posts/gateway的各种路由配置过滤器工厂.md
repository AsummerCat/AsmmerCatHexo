---
title: gateway的各种路由配置过滤器工厂
date: 2019-10-08 16:39:42
tags: [gateway,springCloud,微服务]
---

atewayFilter Factory 是 [Spring Cloud](http://c.biancheng.net/spring_cloud/) Gateway 中提供的过滤器工厂。[Spring](http://c.biancheng.net/spring/) Cloud Gateway 的路由过滤器允许以某种方式修改传入的 HTTP 请求或输出的 HTTP 响应，只作用于特定的路由。

## AddRequestHeader 过滤器工厂

  通过名称我们可以快速明白这个过滤器工厂的作用是添加请求头。

符合规则匹配成功的请求，将添加 X-Request-Foo：bar 请求头，将其传递到后端服务中，后方服务可以直接获取请求头信息。代码如下所示。  

```java
@GetMapping("/hello")
public String hello(HttpServletRequest request) throws Exception {
    System.err.println(request.getHeader("X-Request-Foo"));
    return "success";
}
```

<!--more-->

## RemoveRequestHeader 过滤器工厂

RemoveRequestHeader 是移除请求头的过滤器工厂，可以在请求转发到后端服务之前进行 Header 的移除操作。

```
spring:
  cloud:
    gateway:
      routes:
  - id: removerequestheader_route
  uri: http://c.biancheng.net
    - RemoveRequestHeader=X-Request-Foo
```

## SetStatus 过滤器工厂

SetStatus 过滤器工厂接收单个状态，用于设置 Http 请求的响应码。它必须是有效的 Spring Httpstatus（org.springframework.http.HttpStatus）。它可以是整数值 404 或枚举类型 NOT_FOUND。

```java
spring:
  cloud:
    gateway:
      routes:
        - id: setstatusint_route
  uri: http://c.biancheng.net
  filters:
    - SetStatus=401
```

## RedirectTo过滤器工厂

RedirectTo 过滤器工厂用于重定向操作，比如我们需要重定向到百度。

```java

57JWT是什么
58创建统一认证服务
59服务提供方进行调用认证
60服务消费方申请Token
61Feign调用前统一申请Token传递到调用的服务中
62RestTemplate调用前统一申请Token传递到调用的服务中
63Zuul中传递Token到路由的服务中
64Spring Boot Admin
65Spring Boot Admin开启认证
66Spring Boot Admin集成Eureka
67Spring Boot Admin监控告警服务
68Swagger介绍及使用
69Swagger注解
70Eureka控制台快速查看Swagger文档
71Zuul聚合多个服务Swagger
72微服务实现用户认证
73Spring Cloud服务限流详解
74Spring Cloud服务降级
75灰度发布原理和实现
76Guava Cache本地缓存
77Spring Cloud集成Spring Data Redis
78防止缓存穿透方案
79防止缓存雪崩方案
 首页 > Spring Cloud
阅读：147
Spring Cloud Gateway过滤器工厂的使用
< Gateway路由断言工厂Gateway全局过滤器 >


C语言中文网推出辅导班啦，包括「C语言辅导班、C++辅导班、算法/数据结构辅导班」，全部都是一对一教学：一对一辅导 + 一对一答疑 + 布置作业 + 项目实践 + 永久学习。QQ在线，随时响应！

GatewayFilter Factory 是 Spring Cloud Gateway 中提供的过滤器工厂。Spring Cloud Gateway 的路由过滤器允许以某种方式修改传入的 HTTP 请求或输出的 HTTP 响应，只作用于特定的路由。

Spring Cloud Gateway 中内置了很多过滤器工厂，直接采用配置的方式使用即可，同时也支持自定义 GatewayFilter Factory 来实现更复杂的业务需求。
spring:
  cloud:
    gateway:
      routes:
        - id: add_request_header_route
  uri: http://c.biancheng.net
  filters:
    - AddRequestHeader=X-Request-Foo, Bar

接下来为大家介绍几个常用的过滤器工厂类。
1. AddRequestHeader 过滤器工厂
通过名称我们可以快速明白这个过滤器工厂的作用是添加请求头。

符合规则匹配成功的请求，将添加 X-Request-Foo：bar 请求头，将其传递到后端服务中，后方服务可以直接获取请求头信息。代码如下所示。
@GetMapping("/hello")
public String hello(HttpServletRequest request) throws Exception {
    System.err.println(request.getHeader("X-Request-Foo"));
    return "success";
}
2. RemoveRequestHeader 过滤器工厂
RemoveRequestHeader 是移除请求头的过滤器工厂，可以在请求转发到后端服务之前进行 Header 的移除操作。
spring:
  cloud:
    gateway:
      routes:
  - id: removerequestheader_route
  uri: http://c.biancheng.net
    - RemoveRequestHeader=X-Request-Foo

3. SetStatus 过滤器工厂
SetStatus 过滤器工厂接收单个状态，用于设置 Http 请求的响应码。它必须是有效的 Spring Httpstatus（org.springframework.http.HttpStatus）。它可以是整数值 404 或枚举类型 NOT_FOUND。
spring:
  cloud:
    gateway:
      routes:
        - id: setstatusint_route
  uri: http://c.biancheng.net
  filters:
    - SetStatus=401

4. RedirectTo过滤器工厂
RedirectTo 过滤器工厂用于重定向操作，比如我们需要重定向到百度。
spring:
  cloud:
    gateway:
      routes:
        - id: prefixpath_route
  uri: http://c.biancheng.net
  filters:
    - RedirectTo=302, http://baidu.com
```

# 熔断回退实战

置了 HystrixGatewayFilterFactory 来实现路由级别的熔断，只需要配置即可实现熔断回退功能。配置方式如下所示。

````
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>

内置了 HystrixGatewayFilterFactory 来实现路由级别的熔断，只需要配置即可实现熔断回退功能。配置方式如下所示。
- id: user-service
uri: lb://user-service
predicates:
  - Path=/user-service/**
filters:
  - name: Hystrix
args:
  name: fallbackcmd
fallbackUri: forward:/fallback
````

