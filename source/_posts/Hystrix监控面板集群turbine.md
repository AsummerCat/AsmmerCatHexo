---
title: Hystrix监控面板集群turbine
date: 2019-01-14 10:32:27
tags: [SpringCloud,Hystrix,turbine]
---

# 利用Turbine实现控制台的聚合

比如：服务调用次数、服务调用延迟等。但是仅通过Hystrix Dashboard我们只能实现对服务当个实例的数据展现，在生产环境我们的服务是肯定需要做高可用的，那么对于多实例的情况，我们就需要将这些度量指标数据进行聚合。下面，在本篇中，我们就来介绍一下另外一个工具：Turbine。

<!--more-->

# 创建一个项目 导入pom

```
         <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-turbine</artifactId>
         </dependency>
      
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
         </dependency>
      <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-client</artifactId>
      </dependency>
```



# 启动类上开启Turbine注解

- 创建应用主类`TurbineApplication`，并使用`@EnableTurbine`注解开启Turbine。

```
package com.linjing.hystrixturbine;

import org.springframework.boot.SpringApplication;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.turbine.EnableTurbine;

@SpringCloudApplication
@EnableTurbine
public class HystrixTurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(HystrixTurbineApplication.class, args);
    }
}
```



#  添加配置文件 加入eureka和turbine的相关配置

- 在`application.yml`加入eureka和turbine的相关配置，具体如下：

```
spring:
  application:
    name: Hystrix-Turbine
server:
  port: 9091

# 自定义管理服务器地址
management:
  port: 9092
  address: 127.0.0.1

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/

turbine:
#参数指定了需要收集监控信息的服务名
  app-config: Hystrix-Consumer
  #这里需要注意如果报错 改为 cluster-name-expression: new String('default') 
  cluster-name-expression: "default"
  combine-host-port: true

```

**参数说明**

- `turbine.app-config`参数指定了需要收集监控信息的服务名；
- `turbine.cluster-name-expression` 参数指定了集群名称为default，当我们服务数量非常多的时候，可以启动多个Turbine服务来构建不同的聚合集群，而该参数可以用来区分这些不同的聚合集群，同时该参数值可以在Hystrix仪表盘中用来定位不同的聚合集群，只需要在Hystrix Stream的URL中通过`cluster`参数来指定；
- `turbine.combine-host-port`参数设置为`true`，可以让同一主机上的服务通过主机名与端口号的组合来进行区分，默认情况下会以host来区分不同的服务，这会使得在本地调试的时候，本机上的不同服务聚合成一个服务来统计。

在完成了上面的内容构建之后，我们来体验一下Turbine对集群的监控能力。

![](/img/2019-1-14/HystrixTurbine.png)

# 利用mq 使其异步发送 给turbine 实现监控

Spring Cloud在封装Turbine的时候，还实现了基于消息代理的收集实现。所以，我们可以将所有需要收集的监控信息都输出到消息代理中，然后Turbine服务再从消息代理中异步的获取这些监控信息，最后将这些监控信息聚合并输出到Hystrix Dashboard中。通过引入消息代理，我们的Turbine和Hystrix Dashoard实现的监控架构可以改成如下图所示的结构：

![](/img/2019-1-14/HystrixTurbineToMq.png)

## 引入pom

这里主要引入了`spring-cloud-starter-turbine-amqp`依赖，它实际上就是包装了`spring-cloud-starter-turbine-stream`和`pring-cloud-starter-stream-rabbit`。

```
      <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-turbine-amqp</artifactId>
      </dependency>
```

## 启动类加入启用Turbine Stream的配置

- 在应用主类中使用`@EnableTurbineStream`注解来启用Turbine Stream的配置。

```
@Configuration
@EnableAutoConfiguration
@EnableTurbineStream
@EnableDiscoveryClient
public class TurbineApplication {

   public static void main(String[] args) {
      SpringApplication.run(TurbineApplication.class, args);
   }
}
```

## 添加配置文件

```
spring.application.name=turbine-amqp

server.port=8989
management.port=8990

eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
```



## 然后 在被监控的项目中 加入mq的依赖

使其监控信息能够输出到RabbitMQ上。这个修改也非常简单，只需要在`pom.xml`中增加对`spring-cloud-netflix-hystrix-amqp`依赖，具体如下

```
<dependencies>
   <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-netflix-hystrix-amqp</artifactId>
   </dependency>
</dependencies>
```

在完成了上面的配置之后，我们可以继续之前的所有项目（除turbine以外），并通过Hystrix Dashboard开启对`http://localhost:8989/turbine.stream`的监控，我们可以获得如之前实现的同样效果，只是这里我们的监控信息收集时是通过了消息代理异步实现的。