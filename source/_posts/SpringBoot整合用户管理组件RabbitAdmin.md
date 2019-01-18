---
title: SpringBoot整合用户管理组件RabbitAdmin
date: 2019-01-17 17:36:12
tags: [RocketMQ,消息队列,SpringBoot]
---

# [demo地址](https://github.com/AsummerCat/RabbitmqAdmin)

#  用户管理组件 RabbitAdmin

 RabbitAdmin 类可以很好的操作 rabbitMQ，在 Spring 中直接进行注入即可。

​        !!! 注意，autoStartup 必须设置为 true，否则 Spring 容器不会加载 RabbitAdmin 类。

​    RabbitAdmin 类 的底层实现就是从 Spring 容器中获取 exchange、Bingding、routingkey 以及queue 的 @bean 声明，然后使用 rabbitTemplate 的 execute 方法进行执行对应的声明、修改、删除等一系列 rabbitMQ 基础功能操作。例如添加交换机、删除一个绑定、清空一个队列里的消息等等



<!--more-->



# 编写RabbitMQ配置类

```java
package com.linjing.rabbitmqadmin;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;

/**
 * RabbitMQ的配置文件
 *
 * @author cxc
 * @date 2019/1/17 17:47
 */
@Configuration
public class RabbitMQConfig {

    @Autowired
    private ConnectionFactory connectionFactory;


    /**
     * 管理工具
     *
     * @param connectionFactory
     * @return
     */
    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
        //设置自动开启 默认是ture
        rabbitAdmin.setAutoStartup(true);
        //创建交换机
        rabbitAdmin.declareExchange(new DirectExchange("test.direct", false, false));
        rabbitAdmin.declareExchange(new TopicExchange("topic.direct", false, false));
        rabbitAdmin.declareExchange(new FanoutExchange("fanout.direct", false, false));
        //创建队列
        rabbitAdmin.declareQueue(new Queue("test.direct.queue", false));
        rabbitAdmin.declareQueue(new Queue("test.topic.queue", false));
        rabbitAdmin.declareQueue(new Queue("test.fanout.queue", false));

        //绑定 ->交换机->路由
        rabbitAdmin.declareBinding(new Binding("test.direct.queue",
                Binding.DestinationType.QUEUE, "test.direct", "direct", new HashMap<>()));

        //绑定 ->交换机 ->指定的路由key
        rabbitAdmin.declareBinding(
                BindingBuilder.bind(new Queue("test.topic.queue", false))        //直接创建队列
                        .to(new TopicExchange("topic.direct", false, false))    //直接创建交换机 建立关联关系
                        .with("user.#"));    //指定路由Key

        //绑定 ->交换机
        rabbitAdmin.declareBinding(
                BindingBuilder
                        .bind(new Queue("test.fanout.queue", false))
                        .to(new FanoutExchange("fanout.direct", false, false)));

        //进行解绑
        //rabbitAdmin.removeBinding(BindingBuilder.bind(new Queue("info")).
        //        to(new TopicExchange("direct.exchange")).with("key.2"));
        //
        ////exchange与exchange绑定
        //rabbitAdmin.declareBinding(new Binding("exchange1", Binding.DestinationType.EXCHANGE,
        //        "exchange2", "key.4", new HashMap()));


        //清空队列数据 后面参数是是否异步处理
        rabbitAdmin.purgeQueue("test.topic.queue", false);


        // 删除交换器
        rabbitAdmin.deleteExchange("ArgumentsExchange");
        rabbitAdmin.deleteExchange("alternateExchange");

        //删除队列
        rabbitAdmin.deleteQueue("hello");
        rabbitAdmin.deleteQueue("alternateQueue");
        rabbitAdmin.deleteQueue("basic_queue");
        rabbitAdmin.deleteQueue("dieQueue");


        return rabbitAdmin;
    }


    /** ======================== 定制一些处理策略 =============================*/

    /**
     * 定制化amqp模版 发布确认模式
     * <p>
     * ConfirmCallback接口用于实现消息发送到RabbitMQ交换器后接收ack回调   即消息发送到exchange  ack
     * ReturnCallback接口用于实现消息发送到RabbitMQ 交换器，但无相应队列与交换器绑定时的回调  即消息发送不到任何一个队列中  ack
     */
    @Bean
    public RabbitTemplate rabbitTemplate() {
        Logger log = LoggerFactory.getLogger(RabbitTemplate.class);
        RabbitTemplate rabbitTemp = new RabbitTemplate(connectionFactory);
        // 消息发送失败返回到队列中, yml需要配置 publisher-returns: true
        rabbitTemp.setMandatory(true);

        // 消息返回, yml需要配置 publisher-returns: true
        rabbitTemp.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            String correlationId = message.getMessageProperties().toString();
            log.info("消息：{} 发送失败, 应答码：{} 原因：{} 交换机: {}  路由键: {}", correlationId, replyCode, replyText, exchange, routingKey);
        });

        // 消息确认, yml需要配置 publisher-confirms: true
        rabbitTemp.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("消息发送到exchange成功,id: {}", correlationData.getId());
            } else {
                log.info("消息发送到exchange失败,原因: {}", cause);
            }
        });
        return rabbitTemp;
    }

}

```



# RabbitAdmin

* 该类封装了对 RabbitMQ 的管理操作

```

    @Autowired
    private ConnectionFactory connectionFactory;
@Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
         //设置自动开启 默认是ture
        rabbitAdmin.setAutoStartup(true);
        }
```

* Exchange 操作

* ```java
  //创建四种类型的 Exchange，均为持久化，不自动删除
  rabbitAdmin.declareExchange(new DirectExchange("direct.exchange",true,false));
  rabbitAdmin.declareExchange(new TopicExchange("topic.exchange",true,false));
  rabbitAdmin.declareExchange(new FanoutExchange("fanout.exchange",true,false));
  rabbitAdmin.declareExchange(new HeadersExchange("header.exchange",true,false));
  //删除 Exchange
  rabbitAdmin.deleteExchange("header.exchange");
  ```

* Queue 操作

* ```java
  //定义队列，均为持久化
  rabbitAdmin.declareQueue(new Queue("debug",true));
  rabbitAdmin.declareQueue(new Queue("info",true));
  rabbitAdmin.declareQueue(new Queue("error",true));
  //删除队列     
  rabbitAdmin.deleteQueue("debug");
  //将队列中的消息全消费掉
  rabbitAdmin.purgeQueue("info",false);
  
  ```

  * Binding 绑定

    ```java
    //绑定队列到交换器，通过路由键
    rabbitAdmin.declareBinding(new Binding("debug",Binding.DestinationType.QUEUE,
            "direct.exchange","key.1",new HashMap()));
    
    rabbitAdmin.declareBinding(new Binding("info",Binding.DestinationType.QUEUE,
            "direct.exchange","key.2",new HashMap()));
    
    rabbitAdmin.declareBinding(new Binding("error",Binding.DestinationType.QUEUE,
            "direct.exchange","key.3",new HashMap()));
    
    //进行解绑
    rabbitAdmin.removeBinding(BindingBuilder.bind(new Queue("info")).
            to(new TopicExchange("direct.exchange")).with("key.2"));
    
    //使用BindingBuilder进行绑定
    rabbitAdmin.declareBinding(BindingBuilder.bind(new Queue("info")).
            to(new TopicExchange("topic.exchange")).with("key.#"));
    
    //声明topic类型的exchange 
    rabbitAdmin.declareExchange(new TopicExchange("exchange1",true,false));
    rabbitAdmin.declareExchange(new TopicExchange("exchange2",true,false));
    
    //exchange与exchange绑定
    rabbitAdmin.declareBinding(new Binding("exchange1",Binding.DestinationType.EXCHANGE,
            "exchange2","key.4",new HashMap()));
    
    ```

    ### 接收消息

    - receive（返回 Message 对象）

    ```
    // 接收来自指定队列的消息，并设置超时时间
    Message msg = rabbitTemplate.receive("debug",2000l);
    ```

    - receiveAndConvert（将返回 Message 转换成 Java 对象）

    ```java
    User user = (User) rabbitTemplate.receiveAndConvert();
    ```