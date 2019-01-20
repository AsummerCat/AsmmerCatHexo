---
title: SpringCloudStream整合RabbitMQ使用一
date: 2019-01-18 11:38:33
tags: [springCloud,RabbitMQ,springCloudStream]
---

[文档地址](https://springcloud.cc/spring-cloud-dalston.html#_spring_cloud_stream)

#  SpringCloudStream

SpringCloudStream 就是使用了基于消息系统的微服务处理架构。对于消息系统而言一共分为两类：基于应用标准的 JMS、基于协议标准的 AMQP，在整个 SpringCloud 之中支持有 RabbitMQ、Kafka 组件的消息系统。利用 SpringCloudStream 可以实现更加方便的消息系统的整合处理，但是推荐还是基于 RabbitMQ 实现会更好一些。

 利用消息驱动 bean 的模式可以简化用户的操作复杂度，直接传递一些各类的数据即可实现业务的处理操作。

于是在 SpringBoot 的之中为了方便开发者去整合消息组件，也提供有一系列的处理支持，但是如果按照这些方式来在 SpringCloud 之中进行消息处理，有些人会认为比较麻烦，所以在 SpringCloud 里面将消息整合的处理操作进行了进一步的抽象操作， 实现了更加简化的消息处理。

总结：SpringCloudStream 就是实现了 MDB 功能，同时可以更加简化方便的整合消息组件。

<!--more-->

# 参数 参考

- Barista接口：Barista接口是定义来作为后面类的参数，这一接口定义通道类型和通道名称，`通道名称`是作为`配置用`，`通道类型`则`决定`了app会使用这一通道进行`发送消息`还是从中`接收消息`。
- @Output: 输出注解，用于定义发送消息接口
- @Input: 输入注解，用于定义消息的消费者接口
- @StreamListener: 用于定义监听方法的注解

使用Spring Cloud Stream 非常简单，只需要使用好3个注解即可，在实现高性能消息的生成和消费场景非常合适，但是使用Spring Cloud Stream框架有一个非常大的问题就是`不能实现可靠性投递`，也就是没法保证消息的100%可靠性，会存在少量`消息丢失`问题。

这个原因是因为SpringCloudStream框架`为了和Kafka兼顾`所有在实际工作中使用它的目的就是针对`高性能的消息通信`的！这点就是在当前版本SpringCloudStream的定位。

# 开始

自己看文档，springcloud的封装是默认重试3次，3次失败就丢弃消息了。

我们这边开启两个项目一个项目作为消费者 一个作为生产者

实现生产者@Output  

消费者@Input 绑定接口 监听推送消息

@EnableBinding(PdfNotifyChannel.class)

@StreamListener

通道名需要与生产者对应才能接收消息

# 导入相关pom文件

导入 Stream的包 和rabbitMQ的包

```java
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency>
```

或者也可以这么导入

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-rabbit</artifactId>
        </dependency>
        <!-- 这个代表了spring-cloud-stream+spring-cloud-stream-binder-rabbit-->
```



# 配置文件

```java
spring:
  application:
    name: send-server
  rabbitmq:
    host: 112.74.43.136
    port: 5672
    username: cat
    password: cat
server:
  port: 8081
```



# Spring Cloud Stream中默认实现的对输入消息通道绑定的定义



```java
//发送端
public interface Source {

    String OUTPUT = "output"; // 之前所设置的消息发送的管道

    @Output(Source.OUTPUT)
    MessageChannel output();

}

//接收端
public interface Sink {
    String INPUT = "input";

    @Input("input")
    SubscribableChannel input();
}
```

# 消费者 

### 首先创建一个通道接口

```java
public interface Sink {
    String INPUT = "input";

    @Input("input")
    SubscribableChannel input();
}
```

###  绑定通道接口 监听~ 

@EnableBinding(class)

```
public @interface EnableBinding {
    Class<?>[] value() default {};
}
```



```java
package com.linjing.customer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;

@EnableBinding(Sink.class) // 可以理解为是一个消息的发送管道的定义
public class SinkReceiver {

    private static Logger logger = LoggerFactory.getLogger(SinkReceiver.class);

    @StreamListener(Sink.INPUT) //监听~
    public void receive(Object payload) {
        logger.info("Received: " + payload);
    }

}
```

用来监听INPUT 这个消息

```JAVA
2019-01-18 15:11:23.920  INFO 74939 --- [suk3B-3SM6k5w-1] com.linjing.customer.SinkReceiver        : Received: 嘿嘿嘿
```



# 生产者

### 创建一个通道接口

```java
package com.linjing.producer.controller;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;
import org.springframework.stereotype.Component;

@Component
public interface SinkSender {

    String OUTPUT = "input";

    @Output(SinkSender.OUTPUT)
    MessageChannel output();

}

```

### 绑定通道接口

```java
package com.linjing.producer.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author cxc
 * @date 2019/1/18 14:05
 */
@RestController
@EnableBinding(SinkSender.class)
public class SendController {
    @Autowired
    private SinkSender sinkSender;


    /**
     * 发送消息
     */
    @RequestMapping("send")
    public String send() {
        sinkSender.output().send(MessageBuilder.withPayload("嘿嘿嘿").build());
        return "发送成功";
    }
}

```

### 发送消息

* 注入 通道接口类
*  sinkSender.output().send(MessageBuilder.withPayload("嘿嘿嘿").build());



这样一个简单的实现就完成了



### @SendTo(Processor.OUTPUT) 这里可以消费后 返回

```java
@EnableBinding(Processor.class)
public class TransformProcessor {

  @Autowired
  VotingService votingService;

  @StreamListener(Processor.INPUT)
  @SendTo(Processor.OUTPUT)
  public VoteResult handle(Vote vote) {
    return votingService.record(vote);
  }
}
```