---
title: zuul构建服务网关一
date: 2019-01-15 14:38:25
tags: [SpringCloud,zuul]
---

# 构建服务网关

使用Spring Cloud Zuul来构建服务网关的基础步骤非常简单，只需要下面几步：

<!--more-->

- 创建一个基础的Spring Boot项目，命名为：zuulTest`。并在`pom.xml`中引入依赖：

```java
  <dependency>
            <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zuul</artifactId>
    </dependency>
```



# 创建应用主类，并使用`@EnableZuulProxy`注解开启Zuul的功能。

 

```java
package com.linjing.zuulserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableZuulProxy
@EnableDiscoveryClient
public class ZuulServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulServerApplication.class, args);
    }
}
```



到这里，一个基于Spring Cloud Zuul服务网关就已经构建完毕。启动该应用，一个默认的服务网关就构建完毕了。由于Spring Cloud Zuul在整合了Eureka之后，具备默认的服务路由功能，即：当我们这里构建的`api-gateway`应用启动并注册到eureka之后，服务网关会发现上面我们启动的两个服务`eureka-client`和`eureka-consumer`，这时候Zuul就会创建两个路由规则。每个路由规则都包含两部分，一部分是外部请求的匹配规则，另一部分是路由的服务ID。针对当前示例的情况，Zuul会创建下面的两个路由规则：

- 转发到`eureka-client`服务的请求规则为：`/eureka-client/**`
- 转发到`eureka-consumer`服务的请求规则为：`/eureka-consumer/**`