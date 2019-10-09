---
title: gateway的使用
date: 2019-10-06 09:14:59
tags: [gateway,springCloud,微服务]
---

# 创建gateway网关项目

[demo地址](https://github.com/AsummerCat/gateway-demo)

## 这边的话使用gradle创建项目

```java
主要配置差别就是
dependencies {
    implementation('org.springframework.cloud:spring-cloud-starter-gateway')
}

```

添加了一个gateway的包

如何再添加一个starter-web的包 好像会冲突

里面已经自带了一个fluxweb的包

<!--more-->

# 两种配置方案

## 代码

```java
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

## 配置文件

```java
spring:
  application:
    name: gateway-api
  #可以根据请求参数,cookie,host,请求时间,请求头等进行校验判断路由, 下面根据先后顺序转发
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true

      routes:
      - id: aaa
        uri: http://localhost:9090
        predicates:
          - Path=/helloword/**
        filters:
          - StripPrefix=1 # helloword 转发


server:
  port: 8090

```

**注意**：Gateway默认转发是全路径的，设置`StripPrefix=1`表示从二级url路径转发，即`http://localhost:port/activity/test`将会转发到`http://{activity}/test`