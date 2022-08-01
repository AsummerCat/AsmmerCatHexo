---
title: SpringCloudStream整合RabbitMQ使用二
date: 2019-01-18 15:19:00
tags: [SpringCloud,RabbitMQ,SpringCloudStream]
---

上一篇文章介绍了如何实现一个简单的Stream

[demo地址](https://github.com/AsummerCat/StreamAndRabbitmq)

<!--more--> 

### @SendTo(Processor.OUTPUT) 这个表示将返回值发送出去

```

package com.linjing.customer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;

@EnableBinding(Sink.class)
public class SinkReceiver {

    private static Logger logger = LoggerFactory.getLogger(SinkReceiver.class);

    @StreamListener(Sink.INPUT)
    @SendTo(Processor.OUTPUT)    
    public String receive(String payload) {
        logger.info("Received: " + payload);
        return "谢谢";
    }
}
```

使用`@StreamListener`具有调度条件的示例可以在下面看到。在此示例中，带有值为`foo`的标题`type`的所有消息将被分派到`receiveFoo`方法，所有带有值为`bar`的标题`type`的消息将被分派到`receiveBar`方法。

```
@EnableBinding(Sink.class)
@EnableAutoConfiguration
public static class TestPojoWithAnnotatedArguments {

    @StreamListener(target = Sink.INPUT, condition = "headers['type']=='foo'")
    public void receiveFoo(@Payload FooPojo fooPojo) {
       // handle the message
    }

    @StreamListener(target = Sink.INPUT, condition = "headers['type']=='bar'")
    public void receiveBar(@Payload BarPojo barPojo) {
       // handle the message
    }
}
```

### 轮询发送

```
// 定时轮询发送消息到 binding 为 Processor.OUTPUT
    @Bean
    @InboundChannelAdapter(value = Processor.OUTPUT, poller = @Poller(fixedDelay = "3000", maxMessagesPerPoll = "1"))
    public MessageSource<String> timerMessageSource() {
        return () -> MessageBuilder.withPayload("短消息-" + new Date()).build();
    }

```



来来来 开始正文

# 自定义通道开启

### 首先先看一下配置文件

#### 生产者

```
server:
  port: 8082

spring:
  application:
    name: receive-server
  cloud:
   stream:
     bindings: #多个通道
      firstMe:  # 这个名字是一个通道的名称
        destination: firstMeExchange # 表示要使用的Exchange名称定义
        context-type: text/plain # 设置消息类型，本次为对象json   application/json ，如果是文本则设置“text/plain”
        group: addProductHandler      # 拥有 group 默认会持久化队列

  rabbitmq:
    host: 112.74.43.136
    port: 5672
    username: cat
    password: cat


```

这里需要注意的是 因为生产者的通道交换机是定义的 所以消费者也需要设置

#### 消费者

destination: firstMeExchange # 表示要使用的Exchange名称定义

```
server:
  port: 8082

spring:
  application:
    name: receive-server
  cloud:
   stream:
     bindings: #多个通道
      firstMe:  # 这个名字是一个通道的名称
        destination: firstMeExchange # 表示要使用的Exchange名称定义
        context-type: text/plain # 设置消息类型，本次为对象json   application/json ，如果是文本则设置“text/plain”
        
  rabbitmq:
    host: 112.74.43.136
    port: 5672
    username: cat
    password: cat
```



### 分组与持久化

上述自定义的接口配置中，Spring Cloud Stream 会在 RabbitMQ 中创建一个临时的队列，程序关闭，对应的连接关闭的时候，该队列也会消失。而在实际使用中，我们需要一个持久化的队列，并且指定一个分组，用于保证应用服务的缩放。

只需要在消费者端的 binding 添加配置项 spring.cloud.stream.bindings.[channelName].group = XXX 。对应的队列就是持久化，并且名称为：mqTestOrder.XXX。

```
spring:
  application:
    name: receive-server
  cloud:
   stream:
     bindings: #多个通道
      firstMe:  # 这个名字是一个通道的名称
        destination: firstMeExchange # 表示要使用的Exchange名称定义
        context-type: text/plain # 设置消息类型，本次为对象json   application/json ，如果是文本则设置“text/plain”
        group: addProductHandler      # 拥有 group 默认会持久化队列


```

## rabbitMQ routing key 绑定

```
spring:
  cloud:
    stream:
      bindings:
        inputProductAdd:
          destination: mqTestProduct
          group: addProductHandler      # 拥有 group 默认会持久化队列
        outputProductAdd:
          destination: mqTestProduct
      rabbit:
        bindings:
          inputProductAdd:  #发送
            consumer:
              bindingRoutingKey: addProduct.*       # 用来绑定消费者的 routing key
          outputProductAdd: #消费
            producer:
              routing-key-expression: '''addProduct.*'''  # 需要用这个来指定 RoutingKey
```

spring.cloud.stream.rabbit.bindings.[channelName].consumer.bindingRoutingKey
指定了生成的消息队列的routing key

spring.cloud.stream.rabbit.bindings.[channelName].producer.routing-key-expression 指定了生产者消息投递的routing key

## DLX 队列

DLX:Dead-Letter-Exchange（死信队列）。利用DLX, 当消息在一个队列中变成死信（dead message）之后，它能被重新publish到另一个Exchange，这个Exchange就是DLX。消息变成死信一向有一下几种情况：

消息被拒绝（basic.reject/ basic.nack）并且requeue=false
消息TTL过期（参考：[RabbitMQ之TTL（Time-To-Live 过期时间）](http://blog.csdn.net/u013256816/article/details/54916011)）
队列达到最大长度

DLX也是一个正常的Exchange，和一般的Exchange没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性，当这个队列中有死信时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列，可以监听这个队列中消息做相应的处理。

### Spring Cloud Stream 中使用

```
spring.cloud.stream.rabbit.bindings.[channelName].consumer.autoBindDlq=true

spring.cloud.stream.rabbit.bindings.[channelName].consumer.republishToDlq=true
```

配置说明，可以参考 [spring cloud stream rabbitmq consumer properties](http://docs.spring.io/spring-cloud-stream/docs/Chelsea.SR2/reference/htmlsingle/index.html#_rabbitmq_consumer_properties)

# 结论

Spring Cloud Stream 最大的方便之处，莫过于抽象了事件驱动的一些概念，对于消息中间件的进一步封装，可以做到代码层面对中间件的无感知，甚至于动态的切换中间件，切换topic。使得微服务开发的高度解耦，服务可以关注更多自己的业务流程。

