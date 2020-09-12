---
title: rabbitMQ笔记Exchange交换机(二)
date: 2020-09-12 14:36:00
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记Exchange交换机(二)

## Exchange交换机
常用的三种交换机
```
1.direct (发布与订阅 完全匹配) 也就是点对点

2.fanout (广播)

3.topic (主题,规则匹配)

如果没有绑定交换机怎么办呢？ 没有绑定交换机的话， 消息会发给rabbitmq默认的交换机
里面 默认的交换机隐式的绑定了所有的队列，默认的交换机类型是direct 路由建就是队列的名字
```
<!--more-->
## 交换器和队列的关系

 交换器是通过`路由键`和`队列`绑定在一起的,如果消息拥有`路由键`跟`队列`和`交换机`的路由键匹配,name消息就会被路由到该绑定的队列中.  

 也就是说,消息到队列的过程中,消息首先会经过交换器,接下来交换器在通过路由键匹配分发消息到具体的对列中.  
  路由键可以理解为匹配的规则

---

# direct交换器 发布订阅
发布订阅 点对点
根据路由键投递到不同的queue
```
Direct模式相当于一对一模式,一个消息被发送者发送后,会被转发到一个绑定的消息队列中,然后被一个接收者接收!

直接投递一个请求 一个消费者接收
```
## direct生产者写法
```
   @Autowired
    private RabbitMessagingTemplate rabbitMessagingTemplate;
    public void send(String msg){
        rabbitMessagingTemplate.convertAndSend("交换机","路由键","消息体");
    }
```

---

# topic交换器 主题模式 匹配key
根据路由key的规则进行分发数据   
比如 *.loginfo  
只要是后缀相同的就会发送到同一个queue中

## 写法
```
生产者:
    @Autowired
    private RabbitMessagingTemplate rabbitMessagingTemplate;
    public void send(String msg){
        rabbitMessagingTemplate.convertAndSend("交换机","*.log (路由键模糊匹配)","消息体");
    }
    
消费者:
接收消息 基于类
/**
 * @author cxc
 * @RabbitListener  bindings:绑定队列
 * @Queue 队列
 *        autoDelete:是不是一个临时队列
 * @exchange 指定交换机
 *     type: 指定一个交换机类型 topic模式
 *     key: 指定路由键
 * @date 2019/1/15 20:37
 */
@Component
//标明当前类监听hello这个队列
@RabbitListener(bindings =@QueueBinding(
        value = @Queue(value = RabbitConfig.BASIC_QUEUE, autoDelete = "true"),
        exchange =@Exchange(value = "direct_exchange",type = ExchangeTypes.TOPIC),key = "a.log"))
public class ReceiveHello {
    //表示具体处理的方法
    @RabbitHandler
    public void receive(Message message, Channel channel) {
        System.out.println("处理1" + message);
    }
}
```



---

# fanout交换器 (广播)
只要该交换器下的指定key的queue 都会收到消息
## 写法
因为广播   
所以不需要路由键
```
消费者
@Component
//标明当前类监听hello这个队列
@RabbitListener(bindings =@QueueBinding(
        value = @Queue(value = RabbitConfig.BASIC_QUEUE, autoDelete = "true"),
        exchange =@Exchange(value = "direct_exchange",type = ExchangeTypes.FANOUT)))
public class ReceiveHello {
    //表示具体处理的方法
    @RabbitHandler
    public void receive(String name) {
        System.out.println("处理1" + name);
    }
}
```

```
生产者:
因为广播 不需要路由键所以为""
 @Autowired
    private RabbitMessagingTemplate rabbitMessagingTemplate;
    public void send(String msg){
        rabbitMessagingTemplate.convertAndSend("交换机","","消息体");
    }
```

# 注意点
### 还可以给你的交换机定义属性
比如: 可以设置备用交换机  
```
原先交换机无法被路由到之后,则将其发送到指定的替代交换机

添加 alternate-exchange参数 指定备用交换机
```

# 接收消息

注意: 因为是注解可以直接用配置文件的写法
`"${配置文件配置的queue}"  直接等于字符串`
## 接收消息 基于方法
```
基于方法:
1.接收多个队列
    @RabbitListener(queues = {"firstQueue", "secondQueue"})
    public void receiveFirstQueue(String name) {
        System.out.println("fanout交换机分配请求" + name);
    }


2.接收单个队列
    @RabbitListener(queues = RabbitConfig.TOPIC_QUEUE1)
    public void receiveTopicQueue(String name) {
        System.out.println("topic1交换机分配请求" + name);
    }

3.消息回调确认 ACK机制
    @RabbitListener(queues = RabbitConfig.BASIC_QUEUE)
    public void process(Message message, Channel channel) throws IOException {
        // 采用手动应答模式, 手动确认应答更为安全稳定
        System.out.println("客户端接收到请求");
        try {
            channel.basicQos(1);
            Thread.sleep(3000);
            //手动ack 告诉服务器收到这条消息 已经被我消费了 可以在队列删掉 这样以后就不会再发了 否则消息服务器以为这条消息没处理掉 后续还会在发
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("receive: " + new String(message.getBody()));

            //int i = 1 / 0;

        } catch (Exception e) {
            e.printStackTrace();
            //重新放入队列中
            //channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
            //丢弃这条消息
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
            //拒绝单条消息 重新丢回队列中
            //channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
            System.out.println("receiver fail");
        }
    }

```

## 接收消息 基于类:
```
接收消息 基于类
/**
 * @author cxc
 * @RabbitListener  bindings:绑定队列
 * @Queue 队列
 *        autoDelete:是不是一个临时队列
 * @exchange 指定交换机
 *     type: 指定一个交换机类型
 *     key: 指定路由键
 * @date 2019/1/15 20:37
 */
@Component
//标明当前类监听hello这个队列
@RabbitListener(bindings =@QueueBinding(
        value = @Queue(value = RabbitConfig.BASIC_QUEUE, autoDelete = "true"),
        exchange =@Exchange(value = "direct_exchange",type = ExchangeTypes.DIRECT),key = "路由键"))
public class ReceiveHello {
    //表示具体处理的方法
    @RabbitHandler
    public void receive(Message message, Channel channel) {
        System.out.println("处理1" + message);
    }
}
```

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

 