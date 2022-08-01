---
title: SpringBoot整合rabbitMq
date: 2019-01-15 20:29:18
tags: [SpringBoot,RabbitMQ]
---

# [demo地址](https://github.com/AsummerCat/SpringBootAndRabbitMQ)

# SpringBoot整合rabbitMQ

在Spring Boot中整合RabbitMQ是一件非常容易的事，因为之前我们已经介绍过Starter POMs，其中的AMQP模块就可以很好的支持RabbitMQ

<!--more-->

# 首先先了解下RabbitMQ的消息发送接收的类型

Exchange 类型

Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。只说前三种模式。

1.Direct模式 (直接发送  key对应就可以获取)

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配

2.Topic模式

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“#”和符号“*”。#匹配0个或多个单词，*匹配一个单词。

3.Fanout模式

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。

# 导入RabbitMQ的pom

`spring-boot-starter-amqp`用于支持RabbitMQ。

```
 <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
```



# 配置文件中填写RabbitMq的配置连接信息

```
server:
  port: 9001

spring:
  application:
    name:  boot-rabbitMq
  rabbitmq:
    host: 112.74.43.136
    port: 5672
    username: cat
    password: cat
    
```



# 在项目中创建一个bean  RabbitConfig

- 创建RabbitMQ的配置类`RabbitConfig`，用来配置队列、交换器、路由等高级信息。这里我们以入门为主，先以最小化的配置来定义，以完成一个基本的生产和消费过程。
- 

```
package com.linjing.springbootandrabbitmq;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

    @Bean
    public Queue helloQueue() {
        return new Queue("hello");
    }

}
```



# 直接运行启动类 

你会发现在启动的时候回产生这么一条信息 说明你创建一个新的连接 到rabbitMQ服务器了

```
2019-01-15 20:49:31.414  INFO 32206 --- [           main] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory#39109136:0/SimpleConnection@13275d8 [delegate=amqp://cat@112.74.43.136:5672/, localPort= 57736]

```



#AmqpTemplate Spring实现mq的模板

```
@Autowired
private AmqpTemplate rabbitTemplate;
```

类似于jdbcTemplate redisTemplate 这类的 Spring帮你规定了一些方法 直接调用就可以了 不用管是哪种mq 只要他实现了

amqp规范 就可以使用了

# RabbitMQ发送请求(queue点对点)

```
package com.linjing.springbootandrabbitmq.controller;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;

/**
 * @author cxc
 * @date 2019/1/15 18:01
 */
@RestController
public class SendController {
    @Autowired
    private AmqpTemplate rabbitTemplate;

    @RequestMapping("send")
    public String send() {
        String context = "hello " + new Date();
        System.out.println("Sender : " + context);
        //发送请求
        rabbitTemplate.convertAndSend("hello", context);
        return "发送消息成功";
    }
}

```



# RabbitMQ接收请求(queue点对点)

两种写法

因为事先了amqp规范 所以可以利用@RabbitListener(queues = "") 用来监听消息 

#### @RabbitListener 配合 @RabbitHandler

```
package com.linjing.springbootandrabbitmq.controller;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author cxc
 * @date 2019/1/15 20:37
 */
@Component
//标明当前类监听hello这个队列
@RabbitListener(queues = "hello")
public class ReceiveHello {

    //表示具体处理的方法
    @RabbitHandler
    public void receive(String name) {
        System.out.println("处理1" + name);
    }
}

```

这里需要注意的是: 如果一个监听队列 下有两个处理消息的方法 会报错

```
Caused by: org.springframework.amqp.AmqpException: Ambiguous methods for payload type: class java.lang.String: receive and receiv1e
```

模棱两可的处理方法 他就不知道选择哪一个

## 直接使用@RabbitListener

```
package com.linjing.springbootandrabbitmq.controller;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author cxc
 * @date 2019/1/15 20:37
 */
@Component
public class ReceiveHello {

    //表示具体处理的方法
    @RabbitListener(queues = "hello")
    public void receive(String name) {
        System.out.println("处理" + name);
    }
}

```

可以直接对这个方法进行监听 