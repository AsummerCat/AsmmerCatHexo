---
title: 消息总线实现SpringCloudConfig高可用动态刷新灰度发布
date: 2019-01-18 17:30:21
tags: [SpringCloudBus,SpringCloud,SpringCloudConfig,分布式配置中心,RabbitMQ]
---

# 利用消息总线实现 动态刷新所有节点

工具包:SpringCloudBus,SpringCloudConfig,RabbitMQ

# 照旧导入相关pom文件

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

```

<!--more-->

# 修改日志文件

加入rabbitMQ的配置

```
  rabbitmq: # 现在将集成RabbitMQ作为消息服务总线处理
    host: 112.74.43.136  # RabbitMQ主机服务地址
    port: 5672 # RabbitMQ的监听端口
    username: cat # 用户名
    password: cat # 密码
```

完整

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
          git:
            uri: https://github.com/AsummerCat/SpringCloudConfigFileTest
          # 这里是配置git下面的具体目录 会    -> uri地址下的config-server目录中
            search-paths: config-server
          #  password: 如果需要权限访问
          #  username: 如果需要权限访问

  rabbitmq: # 现在将集成RabbitMQ作为消息服务总线处理
    host: 112.74.43.136  # RabbitMQ主机服务地址
    port: 5672 # RabbitMQ的监听端口
    username: cat # 用户名
    password: cat # 密码

server:
  port: 9092

#配置security的账号密码
security:
  basic:
    enabled: true
  user:
    name: user
    password: password

# 注册到eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8100/eureka/
```



# 启动服务观察

启动此微服务信息，而后一定要保证该微服务已经注册到了 Eureka 之中，同时还需要观察出现 的访问路径，此时会出现有一个重要的路径信息：

```
restartedMain] o.s.b.a.e.mvc.EndpointHandlerMapping : Mapped "{[/bus/refresh],methods=[POST]}"
onto public void org.springframework.cloud.bus.endpoint.RefreshBusEndpoint.refresh(java.lang.String)

```

我们可以在控制台中看到如下内容，在启动时候，客户端程序多了一个`/bus/refresh`请求。

![](/img/2019-1-18/bus.png)



整个方案的架构如上图所示，其中包含了Git仓库、Config Server、以及微服务“Service A”的三个实例，这三个实例中都引入了Spring Cloud Bus，所以他们都连接到了RabbitMQ的消息总线上。

当我们将系统启动起来之后，“Service A”的三个实例会请求Config Server以获取配置信息，Config Server根据应用配置的规则从Git仓库中获取配置信息并返回。

此时，若我们需要修改“Service A”的属性。首先，通过Git管理工具去仓库中修改对应的属性值，但是这个修改并不会触发“Service A”实例的属性更新。我们向“Service A”的实例3发送POST请求，访问`/bus/refresh`接口。此时，“Service A”的实例3就会将刷新请求发送到消息总线中，该消息事件会被“Service A”的实例1和实例2从总线中获取到，并重新从Config Server中获取他们的配置信息，从而实现配置信息的动态更新。

而从Git仓库中配置的修改到发起`/bus/refresh`的POST请求这一步可以通过Git仓库的Web Hook来自动触发。由于所有连接到消息总线上的应用都会接受到更新请求，所以在Web Hook中就不需要维护所有节点内容来进行更新，从而解决了通过Web Hook来逐个进行刷新的问题。



# 指定刷新范围

上面的例子中，我们通过向服务实例请求Spring Cloud Bus的`/bus/refresh`接口，从而触发总线上其他服务实例的`/refresh`。但是有些特殊场景下（比如：灰度发布），我们希望可以刷新微服务中某个具体实例的配置。

## 参数 destination

```
/bus/refresh?destination=服务名:9000
```

Spring Cloud Bus对这种场景也有很好的支持：`/bus/refresh`接口还提供了`destination`参数，用来定位具体要刷新的应用程序。比如，我们可以请求`/bus/refresh?destination=customers:9000`，此时总线上的各应用实例会根据`destination`属性的值来判断是否为自己的实例名，若符合才进行配置刷新，若不符合就忽略该消息。

`destination`参数除了可以定位具体的实例之外，还可以用来定位具体的服务。定位服务的原理是通过使用Spring的PathMatecher（路径匹配）来实现，比如：`/bus/refresh?destination=customers:**`，该请求会触发`customers`服务的所有实例进行刷新。