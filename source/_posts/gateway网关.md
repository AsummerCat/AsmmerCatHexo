---
title: gateway网关
date: 2020-05-09 15:11:24
tags: [SpringCloud,gateway]
---

# gateway网关

```
 gateway 非阻塞异步模型 (推荐)
 zuul 同步阻塞模型 
 zuul2 异步非阻塞模型 (还没完全上架)
```

## 工作流程

核心就是: 路由转发+执行过滤链

fiter前后拦截 类似前置通知和后置通知

内容:

```
1.Route(路由)
2.Predicate(断言)
3.Filter(过滤)
```

<!--more-->

## 使用

### 引入相关pom

```
gateway的
```

### 在启动类上加入注解

```
注册中心的注解 @EnableDiscoveryClient
```

## 路由规则配置的方法

### 第一种 yml配置

```
spring:
  application:
    name: gateway-api
  #可以根据请求参数,cookie,host,请求时间,请求头等进行校验判断路由, 下面根据先后顺序转发
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
    # 下面是路由规则
      routes:
       - id: aaa   # 路由的ID 需要保证唯一 建议配合服务名
        uri: http://localhost:9090  # 匹配后提供提供服务的路由地址
        predicates:  
          - Path=/helloword/**   # 断言,路径相匹配的进行路由
        filters:
          - StripPrefix=1 # helloword 转发
          
       - id: test-demo   # 路由的ID 需要保证唯一 建议配合服务名
        uri: http://localhost:8090  # 匹配后提供提供服务的路由地址
        predicates:  
          - Path=/test/**   # 断言,路径相匹配的进行路由
        filters:
          - StripPrefix=1 # test 转发          
```

**注意**：Gateway默认转发是全路径的，设置StripPrefix=1表示从二级url路径转发，即http://localhost:port/activity/test将会转发到http://{activity}/test

### 第二种 硬编码版本

```
package com.linjingc.gatewayapi;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 代码化router
 */
@Configuration
public class GatewayRoutes {
    @Bean
    public RouteLocator routeLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route(r ->
                        r.path("/helloword")
                                .filters(
                                        f -> f.stripPrefix(1)
                                )
                                .uri("http://localhost:9090/helloword")
                )
                .build();
    }
}
```

## 动态路由 这边的话是指整合了注册中心实现的

 两种方式

### 第一种 根据负载均衡自动路由  -> 默认根据服务自动匹配

```
   application:
    name: gateway-rureka

  cloud:
    gateway:
      ## 默认根据服务自动匹配
      discovery:
        locator:
            ## 开启从注册中心动态创建路由的功能
          enabled: true
          ## 服务名称小写
          lowerCaseServiceId: true

```

### 第二种 根据自定义路由规则路由  -> 自定义路由

```
  application:
    name: gateway-rureka

  cloud:
    gateway:
    # 自定义路由
      routes:
        - id: web-server
          # lb: 前缀固定 ->后面是服务名称
          uri: lb://WEB-SERVER
          predicates:
            - Path=/web/**
          filters:
            - StripPrefix=1
```

## redicates 断言重点解析

```
这个就是根据不同的断言进行匹配路径
常见的有上面的 -path
```

###  predicates的断言方法

```
具体内容官网查看 predicates的内容

1.After         在什么时间之前允许访问     可以用ZonedDateTime.now();获取默认时区的时间
2.Before        在什么时候之后允许访问
3.Between       在什么时间区间内允许访问
4.Cookie        要携带什么Cookie才能访问  格式:key,value
5.Header        要携带什么Header才能访问  格式:请求头属性,正则
6.Host          要什么样的Host才能访问    格式: **.linjingc.top
7.Method        什么样的请求方式才能访问   格式:GET  
8.Path         要是怎样的路径才能访问
9.Query         要携带什么样的参数才能访问
```

### 过滤器 

可以参考[linnjingc.top的gateway的各种路由配置过滤器工厂](https://linjingc.top/2019/10/08/gateway%E7%9A%84%E5%90%84%E7%A7%8D%E8%B7%AF%E7%94%B1%E9%85%8D%E7%BD%AE%E8%BF%87%E6%BB%A4%E5%99%A8%E5%B7%A5%E5%8E%82/)

```
可以实现GlobalFilter接口和Orders(用来过滤链排序)来完成 自定义全局过滤器
可以实现GatewayFilter接口来实现 自定义单一的过滤器 (可以直接写在yml文件中)
```



