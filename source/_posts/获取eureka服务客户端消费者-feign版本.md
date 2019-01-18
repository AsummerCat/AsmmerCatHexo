---
title: 获取eureka服务客户端消费者-feign版本
date: 2019-01-10 14:06:48
tags: [springCloud,eureka,feign]
---

#  [demo地址](https://github.com/AsummerCat/springCloudCustomer)

# eureka注册中心 (略)

这个操作简单已经注册成功了  主要是 消费端的使用

## eureka服务端 (略)

这个就是注册个服务到eureka就可以了

<!--more-->

# 导入相关pom文件

```java
 <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
```

## 修改应用主类。通过`@EnableFeignClients`注解开启扫描Spring Cloud Feign客户端的功能

```JAVA
package com.linjing.feigncustomer;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;

/**
 * @author cxc
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignCustomerApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignCustomerApplication.class, args);
    }
}

```



## 创建一个接口用来绑定feign服务

创建一个Feign的客户端接口定义。使用`@FeignClient`注解来指定这个接口所要调用的服务名称，接口中定义的各个函数使用Spring MVC的注解就可以来绑定服务提供方的REST接口，

```java
package com.linjing.feigncustomer.server;

import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * @author cxc
 * @date 2019/1/10 14:47
 */
@Component

//服务名称 注解只能放在类上
@FeignClient("test-server")
public interface TestServer {

    //服务的接口名称映射
    @GetMapping("/test")
    String test();
}
```



## Controller 直接调用

```java
package com.linjing.feigncustomer.controller;

import com.linjing.feigncustomer.server.TestServer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author cxc
 * @date 2019/1/10 14:49
 */
@RestController
public class TestController {
    @Autowired
    private TestServer testServer;

    @RequestMapping("test")
    public String test() {
        return testServer.test();
    }
}
```





这样就完成了  feign 会根据你给的服务名称去rureka 找到服务 进行负载均衡 处理