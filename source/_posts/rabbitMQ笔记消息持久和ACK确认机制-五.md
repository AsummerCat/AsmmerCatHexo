---
title: rabbitMQ笔记消息持久和ACK确认机制(五)
date: 2020-09-12 14:37:52
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记消息持久和ACK确认机制(五)
## 消息队列持久化
```
@RabbitListener在该注解里 
@queue和@Exchange 都不要设置autoDelete="true";
不然就会变成临时的消息 不持久化

@autoDelete
写在:
 @Queue中: 当所有消费客户端断开后,是否自动删除队列
 true:是 false:否

写在:
 @Exchange中: 当所有绑定队列都不在使用时,是否自动删除交换器
 true:是 false:否
```
<!--more-->
不然就会变成临时的 
消费者宕机后 消息不会存在

## RabbitMQ中的ACK消息确认机制
### 什么是消息确认ACK?
如果在处理消息的过程中,消费者的服务器在处理消息时出现异常,那可能这条正在处理的消息就没有完成消息消费,<font color="red">数据就会丢失</font>.为了确保数据不会丢失,RabbitMQ支持消息确认-ACK
### ACK的消息确认机制
ACK机制是消费者从RabbitMQ收到消息并处理完成后,反馈给rabbitMQ,<font color="red">rabbitMQ收到反馈后才将此消息从队列中删除</font>
1. 如果一个消费者在处理消息出现了网络不稳定'服务器异常等现象,name就不会有ACK反馈,rabbitMQ会认为这个消息没有正常消费,<font color="red">会将消息重新返回队列中.</font>
2. 如果在集群的情况下:rabbitMQ会立即将这个消息推送给这个在线的其他消费者.这种机制保证了在消费者服务端故障的时候,不会丢失任何消息和任务
3. <font color="red">消息永远不会从rabbitMQ中删除</font>:只有当消费者正确发送ACK返回,rabbitMQ确认收到后,消息才会从rabbitMQ服务器的数据中删除
4. 消息的ACK确认机制默认是打开的

### ACK机制的开发注意事项
如果忘记了ACK,那么后果很严重.  
当Consumer退出时,Message会一直重新分发.然后rabbitMQ<font color="red">会占用越来越多的内存</font>,由于rabbitMQ会长时间运行,因此这个"内存泄露"是致命的

# 如何解决ACK的无限重发(重点)
1. 在消息接收的方法里,加入`try catch`让程序正常返回
2. 在消费者配置文件里接收消息重试次数
```
# 开启重试
spring.rabbitmq.listener.retry.enable=true
# 重试次数,默认为3次
spring.rabbitmq.listener.retry.max-attempts=5
 



----------------------------
以下部分是 手动ACK方式
#  对于订阅模式采取手动应答
spring.rabbitmq.listener.direct.acknowledge-mode=manual

#  对于主题模式采取手动应答
spring.rabbitmq.listener.topic.acknowledge-mode=manual


# 生产者消息发送到交换机确认机制,是否确认回调
spring.rabbitmq.publisher-confirms=true  
# 交换机发送消息给队列失败后返回确认
spring.rabbitmq.publisher-returns=true

```
# 消费者要如何确认消息
 默认情况下 消费者在拿到rabbitmq的消息时,已经自动确认这条消息已经消费了,讲白话就是rabbitmq的队列里就会删除这条消息了,但是我们实际开发中难免会遇到这种情况   
比如说 拿到这条消息 发
现我处理不了 比如说 参数不对,又比如说 我当前这个系统出问题了,暂时不能处理这个消息， 但是这个消息已经被你消费掉了 rabbitmq的队列里也删除掉了,你自己这边又处理不了,那么,这个消息就被遗弃了。  
这种情况在实际开发中是不合理的,rabbitmq提供了解决这个问题的方案,也就是我们上面所说的confirm模式

```
@Bean
public SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory =
            new SimpleRabbitListenerContainerFactory();
    //这个connectionFactory就是我们自己配置的连接工厂直接注入进来
    simpleRabbitListenerContainerFactory.setConnectionFactory(connectionFactory);
    //这边设置消息确认方式由自动确认变为手动确认
    simpleRabbitListenerContainerFactory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
    return simpleRabbitListenerContainerFactory;
}

或者配置文件设置

spring.rabbitmq.listener.direct.acknowledge-mode=manual

#  对于主题模式采取手动应答
spring.rabbitmq.listener.topic.acknowledge-mode=manual

```
## 关于AcknowledgeMode这个类
```
3个状态 不确认 手动确认 自动确认
        NONE    MANUAL  AUTO
```
## 关于手动确认的消费者写法
```
/**
     * 测试参数队列
     *
     * @param message
     * @param channel
     * @throws IOException
     */

    @RabbitListener(queues = "ArgumentsQueue")
    public void argumentsQueueProcess(Message message, Channel channel) throws IOException {
        // 采用手动应答模式, 手动确认应答更为安全稳定
        System.out.println("测试客户端接收到请求");
        try {
            //正常手动确认 第二个参数是是否批量确认
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("receive: " + new String(message.getBody()));
        } catch (Exception e) {
            e.printStackTrace();
            //发生异常 手动确认消息
            //第三个参数表示是这条消息是返回到原队列 还是这条消息作废 就是不退回了。
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
            System.out.println("receiver fail");
        }
    }
```