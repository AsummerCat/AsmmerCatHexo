---
title: SpringBoot整合rabbitMq的点对点及其广播
date: 2019-01-16 13:57:47
tags: [SpringBoot,RabbitMQ]
---



# [demo地址]()



# [参考文章](https://blog.csdn.net/zhuzhezhuzhe1/article/details/80454956)

# 疑问?

* 如果消息发送 接收失败的时候 消息如何处理?

  如果消息在接收的时候发生了错误 比如 抛出了异常  会无法给rabbitMQ发送

  如果一个消费者在发送确认信息前死去（连接或通道关闭、TCP连接丢失等），RabbitMQ将会认为该消息没有被完全处理并会重新将消息加入队列。如果此时有其他的消费者，RabbitMQ很快就会重新发送该消息到其他的消费者。通过这种方式，你完全可以保证没有消息丢失，即使某个消费者意外死亡。

  对RabbitMQ而言，没有消息超时这一说。如果消费者死去，RabbitMQ将会重新发送消息。即使处理一个消息需要耗时很久很久也没有关系。

  

  <!--more-->

  # 几种基本的发送类型

  ###  default queue 默认 Direct模式 

  Direct模式相当于一对一模式,一个消息被发送者发送后,会被转发到一个绑定的消息队列中,然后被一个接收者接收!

  直接投递一个请求 一个消费者接收

  

  ###   Exchange queue 订阅投递    Fanout模式

   Exchange本身不保存消息 看起来是用于转发消息的

  需要绑定队列  

  根据 routingKey 路由关键字 投递消息 

  如果多个队列绑定同一个routingKey  会投递给所有绑定的队列 

  

  每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。

  

  ### TOPIC 模式

  direct类型的Exchange路由规则是完全匹配binding key与routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。topic类型的Exchange在匹配规则上进行了扩展，它与direct类型的Exchage相似，也是将消息路由到binding key与routing key相匹配的Queue中，但这里的匹配规则有些不同，它约定：

  routing key为一个句点号“. ”分隔的字符串（我们将被句点号“. ”分隔开的每一段独立的字符串称为一个单词），如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”
  binding key与routing key一样也是句点号“. ”分隔的字符串
  binding key中可以存在两种特殊字符“*”与“#”，用于做模糊匹配，其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）】

  ![](/img/2019-1-15/topic.png)


  以上图中的配置为例，routingKey=”quick.orange.rabbit”的消息会同时路由到Q1与Q2，routingKey=”lazy.orange.fox”的消息会路由到Q1，

  routingKey=”lazy.brown.fox”的消息会路由到Q2，routingKey=”lazy.pink.rabbit”的消息会路由到Q2（只会投递给Q2一次，虽然这个routingKey与Q2的两个bindingKey都匹配）；

  routingKey=”quick.brown.fox”、routingKey=”orange”、routingKey=”quick.orange.male.rabbit”的消息将会被丢弃，因为它们没有匹配任何bindingKey。

   

  #  写一个rabbitMQ的配置类 用来存放接收的和发送的消息类型 

```
package com.linjing.springbootandrabbitmq;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * RabbitMQ配置类
 */
@Configuration
public class RabbitConfig {

    /**
     * 基本消息发送
     */
    public static final String BASIC_QUEUE = "basic_queue";


    /**
     * 消息交换机的名字
     */
    public static final String EXCHANGE = "exchangeTest";
    /**
     * 队列key1
     */
    public static final String ROUTINGKEY1 = "queue_one_key1";
    /**
     * 队列key2
     */
    public static final String ROUTINGKEY2 = "queue_one_key2";


    /**
     * TOPIC模式
     */
    public static final String TOPIC_QUEUE1 = "topic.queue1";
    public static final String TOPIC_QUEUE2 = "topic.queue2";
    public static final String TOPIC_EXCHANGE = "topic.exchange";


    /**
     * *********************************************************************************************
     *  ********************************Direct模式普通投递************************************
     *   *********************************************************************************************
     */

    /**
     * 基本点对点 消息发送
     */
    @Bean
    public Queue helloQueue() {
        return new Queue(BASIC_QUEUE);
    }


    /**
     * ***********************************************************************************************
     *  **************************************Fanout模式交换机投递*************************************************
     *   *********************************************************************************************
     */
    /**
     * Fanout模式
     * 消息中的路由键（routing key）如果和 Binding 中的 binding key 一致， 交换器就将消息发到对应的队列中。路由键与队列名完全匹配/**
     *
     * @return * 交换机投递
     */
    @Bean
    public DirectExchange directExchange() {
        DirectExchange directExchange = new DirectExchange(EXCHANGE, true, false);
        return directExchange;
    }

    @Bean
    public Queue firstQueue() {
        return new Queue("firstQueue");
    }

    @Bean
    public Queue secondQueue() {
        return new Queue("secondQueue");
    }

    /**
     * 将消息队列1和交换机进行绑定
     */
    @Bean
    public Binding binding_one() {
        return BindingBuilder.bind(firstQueue()).to(directExchange()).with(ROUTINGKEY1);
    }

    /**
     * 将消息队列2和交换机进行绑定
     */
    @Bean
    public Binding binding_two() {
        return BindingBuilder.bind(secondQueue()).to(directExchange()).with(ROUTINGKEY1);
    }

    /**
     * ***********************************************************************************************
     * **************************************Topic模式 投递*************************************************
     * *********************************************************************************************
     */


    @Bean
    public Queue topicQueue1() {
        return new Queue(TOPIC_QUEUE1);
    }

    @Bean
    public Queue topicQueue2() {
        return new Queue(TOPIC_QUEUE2);
    }

    /**
     * 创建top模式交换机
     *
     * @return
     */
    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(TOPIC_EXCHANGE);
    }

    @Bean
    public Binding topicBinding1() {
        return BindingBuilder.bind(topicQueue1()).to(topicExchange()).with("test.message");
    }

    @Bean
    public Binding topicBinding2() {
        return BindingBuilder.bind(topicQueue2()).to(topicExchange()).with("test.#");
    }


}
```



## 接收消息

```
package com.linjing.springbootandrabbitmq.controller;

import com.linjing.springbootandrabbitmq.RabbitConfig;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

/**
 * @author cxc
 * @date 2019/1/15 20:37
 */
@Component
public class ReceiveMessage {

    /**
     * fanout交换机投递消息 因为有监听了两个队列  然后 两个队列又是绑定的同一个路由 所以 会执行两次
     *
     * @param name
     */
    @RabbitListener(queues = {"firstQueue", "secondQueue"})
    public void receiveFirstQueue(String name) {
        System.out.println("fanout交换机分配请求" + name);
    }


    /**
     * 匹配请求
     * topic
     */
    @RabbitListener(queues = RabbitConfig.TOPIC_QUEUE1)
    public void receiveTopicQueue(String name) {
        System.out.println("topic1交换机分配请求" + name);
    }

    @RabbitListener(queues = RabbitConfig.TOPIC_QUEUE2)
    public void receiveTopicQueue2(String name) {
        System.out.println("topic2交换机分配请求" + name);
    }

}
```

# # 发送消息

```
package com.linjing.springbootandrabbitmq.controller;

import com.linjing.springbootandrabbitmq.RabbitConfig;
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
        String context = "hello基本发送 " + new Date();
        System.out.println("Sender : " + context);
        //发送请求
        rabbitTemplate.convertAndSend(RabbitConfig.BASIC_QUEUE, context);
        return "发送消息成功";
    }

    /**
     * 发送交换机请求
     *
     * @return
     */
    @RequestMapping("sendFanout")
    public String sendExchangeTest() {
        String context = "hello 发送交换机请求" + new Date();
        System.out.println("Sender : " + context);
        //发送请求
        //队列名称 ->路由队列名称 ->内容
        rabbitTemplate.convertAndSend(RabbitConfig.EXCHANGE, RabbitConfig.ROUTINGKEY1, context);
        return "发送消息成功";
    }

    /**
     * 发送订阅请求
     *
     * @return
     */
    @RequestMapping("sendTopic")
    public String sendTopic() {
        String context = "hello 发送topic请求" + new Date();
        System.out.println("Sender : " + context);
        //发送请求
        //队列名称 ->路由队列名称 ->内容
        rabbitTemplate.convertAndSend(RabbitConfig.TOPIC_EXCHANGE, "test.message", context);
        return "发送消息成功";
    }


}
```



# 如果需要回调消息的话(应答)

## 修改配置文件

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
    publisher-confirms: true #  消息发送到交换机确认机制,是否确认回调
    publisher-returns: true  # 消息返回确认
    ##开启ack
    listener:
      direct:
      ## 采取手动应答
        acknowledge-mode: manual
      simple:
        acknowledge-mode: manual
```

改为rabbitMQ为手动应答

## 接着修改控制层

```
将
@Autowired
private AmqpTemplate rabbitTemplate;
修改为
@Autowired
private RabbitTemplate rabbitTemplate;
改为RabbitTemplate自己的实现模板
```

## 添加应答

方式1

```
rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (!ack) {
                System.out.println("HelloSender消息发送失败" + cause + correlationData.toString());
            } else {
                System.out.println("HelloSender 消息发送成功 ");
            }
        });
```

方式2 

重写rabbitTemplate 注入

```
 @Autowired
    private ConnectionFactory connectionFactory;
    
     /**
     * 定制化amqp模版
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
```

ConfirmCallback    ask为true 发送成功     否则     发生的情况交换机挂了 连接失败

ReturnCallback 发生情况 交换机未绑定队列 或者无队列接收请求

需要注意的是如果开启了手动应答 比如应答给服务器 否则会表示为未发送成功 下次还会继续发送该消息 消息会一直堵塞在服务器中



## 发送消息

```
@RequestMapping("send")
public String send() {
    String context = "hello基本发送 " + new Date();
    System.out.println("Sender : " + context);
    //发送请求
    //赋予一个Id 用于确认消息回调
    CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
    rabbitTemplate.convertAndSend(RabbitConfig.BASIC_QUEUE, "", context, correlationData);
    return "发送消息成功";
}
```

## 接收消息

```
   @RabbitListener(queues = RabbitConfig.BASIC_QUEUE)
    public void process(Message message, Channel channel) throws IOException {
        // 采用手动应答模式, 手动确认应答更为安全稳定
        System.out.println("客户端接收到请求");
        try {

            //手动ack 告诉服务器收到这条消息 已经被我消费了 可以在队列删掉 这样以后就不会再发了 否则消息服务器以为这条消息没处理掉 后续还会在发
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("receive: " + new String(message.getBody()));

        } catch (Exception e) {
            e.printStackTrace();
            //重新放入队列中
            //channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
            //丢弃这条消息
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
            System.out.println("receiver fail");
        }
    }
```

需要注意的是 如果 不回调的话 下次还会继续发送

//消息的标识，false只确认当前一个消息收到，true确认所有consumer获得的消息

channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

//ack返回false，并重新回到队列，api里面解释得很清楚

channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);

//拒绝消息

channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);





# 如果需要开启rabbitMQ事务的话

#### 注意：**发布确认和事务。(两者不可同时使用)在channel为事务时，不可引入确认模式；同样channel为确认模式下，不可使用事务**

注意 开启事务的话 是同步的 对吞吐率影响较大

```
修改配置文件 关闭发布确认模式
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
    publisher-confirms: false #  消息发送到交换机确认机制,是否确认回调
    publisher-returns: false  # 消息返回确认
#    ##开启ack
#    listener:
#      direct:
#      ## 采取手动应答
#        acknowledge-mode: manual
#      simple:
#        acknowledge-mode: manual

```

### 接着修改 rabbitMQ开启事务

```
//开启事务
rabbitTemp.setChannelTransacted(true);
```

```
/**
     * 定制化amqp模版 事务模式
     */
    @Bean
    //@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public RabbitTemplate rabbitTemplate() {
        Logger log = LoggerFactory.getLogger(RabbitTemplate.class);
        RabbitTemplate rabbitTemp = new RabbitTemplate(connectionFactory);
        // 消息发送失败返回到队列中, yml需要配置 publisher-returns: true
        rabbitTemp.setMandatory(true);
        //开启事务
        rabbitTemp.setChannelTransacted(true);
        return rabbitTemp;
    }
```

### 发送消息

```
因为整合了SpringBoot  而且在RabbitTemplate也配置了开启事务 
所以我们可以直接使用spring的事务注解
我们通过调用者提供外部事务@Transactional(rollbackFor = Exception.class)，来现实事务方法。一旦方法中抛出异常，比如执行数据库操作时，就会被捕获到，同时事务将进行回滚，并且向外发送的消息将不会发送出去。
package com.linjing.springbootandrabbitmq.controller;

import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * 发送事务消息
 */
@Component
public class TransactionSender2 {

    @Autowired
    private AmqpTemplate rabbitTemplate;

    @Transactional(rollbackFor = Exception.class)
    public void send(String msg) {
        SimpleDateFormat time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String sendMsg = msg + time.format(new Date()) + " This is a transaction message！ ";
        /**
         * 这里可以执行数据库操作
         *
         **/
        System.out.println("TransactionSender2 : " + sendMsg);
        this.rabbitTemplate.convertAndSend("basic_queue", sendMsg);
    }
}
```

###  接收事务

```
 /**
     * 事务模式
     */
    @RabbitListener(queues = RabbitConfig.BASIC_QUEUE)
    public void process(Message message, Channel channel) throws IOException {
        // 采用手动应答模式, 手动确认应答更为安全稳定
        System.out.println("客户端接收到请求");
            //告诉服务器收到这条消息 已经被我消费了 可以在队列删掉 这样以后就不会再发了 否则消息服务器以为这条消息没处理掉 后续还会在发
            //channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("receive: " + new String(message.getBody()));
            //模拟异常 回滚
            //int i=1/0;
    }
```

事务消费失败 会一直重发  直到被消费成功



