---
title: 获取eureka服务客户端消费者-ribbon版本
date: 2019-01-10 11:27:12
tags: [SpringCloud,eureka,ribbon]
---

# eureka注册中心 (略)

这个操作简单已经注册成功了  主要是 消费端的使用

## eureka服务端 (略)

这个就是注册个服务到eureka就可以了

<!--more-->

# 导入相关pom文件

```java
  <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
```



## 初始化RestTemplate

- 创建应用主类。初始化`RestTemplate`，用来真正发起REST请求。`@EnableDiscoveryClient`注解用来将当前应用加入到服务治理体系中。
- @LoadBalanced

当我们使用Spring Cloud Ribbon实现客户端负载均衡的时候，通常都会利用`@LoadBalanced`来让`RestTemplate`具备客户端负载功能

由于`RestTemplate`被`@LoadBalanced`修饰，所以它具备客户端负载均衡的能力，当请求真正发起的时候，url中的服务名会根据负载均衡策略从服务清单中挑选出一个实例来进行访问。

- 修改应用主类。为`RestTemplate`增加`@LoadBalanced`注解

- ```java
  @Bean
  //注解
  @LoadBalanced
  	public RestTemplate restTemplate() {
  		return new RestTemplate();
  	}
  ```





## 修改Cpntroller

去掉原来通过`LoadBalancerClient`选取实例和拼接URL的步骤，直接通过RestTemplate发起请求。

```java
package com.linjing.ribboncustomer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @author cxc
 * @date 2019/1/10 11:12
 */
@RestController
public class RibbonController {
    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/consumer")
    public String consumer() {
        //直接获取注册中心的名称
        String url = "http://test-server/test";
        System.out.println(url);
        return restTemplate.getForObject(url, String.class);
    }
}
```



可以看到这里，我们除了去掉了原来与`LoadBalancerClient`相关的逻辑之外，对于`RestTemplate`的使用，我们的第一个url参数有一些特别。这里请求的host位置并没有使用一个具体的IP地址和端口的形式，而是采用了服务名的方式组成。那么这样的请求为什么可以调用成功呢？因为Spring Cloud Ribbon有一个拦截器，它能够在这里进行实际调用的时候，自动的去选取服务实例，并将实际要请求的IP地址和端口替换这里的服务名，从而完成服务接口的调用。