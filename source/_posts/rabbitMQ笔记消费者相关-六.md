---
title: rabbitMQ笔记消费者相关(六)
date: 2020-09-12 14:38:24
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记消费者相关(六)

# 常规消费消息
### 基于方法
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
<!--more-->
### 基于类
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
# 如何手动确认消息 ACK机制
## 开启手动确认消息
### 第一种 yml
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
```
### 第二种 配置类
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
```
## 手动确认的消费者如何写
```
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

# 消息预取
因为rabbitMQ默认是轮训发送队列给消费者消费的
可能出现这么个情况:  
我打个最简单的比方 有100个订单消息需要处理（消费） 现在有消费者A 和消费者B ， 消费者A消费一条消息的速度是 10ms 消费者B 消费一条消息的速度是15ms （ 当然 这里只是打比方） 那么 rabbitmq 会默认给消费者A B 一人50条消息让他们消费 但是 消费者A 他500ms 就可以消费完所有的消息 并且处于空闲状态 而 消费者B需要750ms 才能消费完 如果从性能上来考虑的话 这100条消息消费完的时间一共是750ms（因为2个人同时在消费） 但是如果 在消费者A消费完的时候 能把这个空闲的性能用来和B一起消费剩下的信息的话， 那么这处理速度就会快非常多。  
<font color="red">消息预取:简单来说就是  
消费者消费前告诉MQ,我一次性需要多少条数据,全部消费完之后,再接着给我发送消息  
注意:在使用消息预取前 要注意一定要设置为手动确认消息</font>
## 第一种添加方式 配置类
```
@Bean
public SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory =
            new SimpleRabbitListenerContainerFactory();
    simpleRabbitListenerContainerFactory.setConnectionFactory(connectionFactory);
    //手动确认消息
    simpleRabbitListenerContainerFactory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
    //设置消息预取的数量
    simpleRabbitListenerContainerFactory.setPrefetchCount(1);
    return simpleRabbitListenerContainerFactory;
}
```
## 第二种 yml
```
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
        ## 批量一次性接收多少条数据
        prefetch:  10
      simple:
        acknowledge-mode: manual
```

# 特别注意一个事情 死信交换机
在消费者确认消息的时候 
有一种情况
```
//丢弃这条消息
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
```
这个消息会被丢弃到哪里去  
如果配置了死信交换机   
<font color="red">这个被丢弃的消息会进去死信交换机</font>  
在创建队列的时候 可以给这个队列附带一个交换机, 那么这个队列作废的消息就会被重新发到附带的交换机,然后让这个交换机重新路由这条消息
### 写法
```
@Bean
    public Queue queue() {
        Map<String,Object> map = new HashMap<>();
        //设置消息的过期时间 单位毫秒
        map.put("x-message-ttl",10000);
        //设置附带的死信交换机
        map.put("x-dead-letter-exchange","exchange.dlx");
        //指定重定向的路由建 消息作废之后可以决定需不需要更改他的路由建 如果需要 就在这里指定
        map.put("x-dead-letter-routing-key","dead.order");
        return new Queue("testQueue", true,false,false,map);
    }
```
在死信交换机中有个队列回去消费该消息
```
@RabbitListener(queues = "dieQueue")
    public void dieQueueProcess(Message message, Channel channel) throws IOException {
        // 采用手动应答模式, 手动确认应答更为安全稳定
        System.out.println("死信客户端接收到请求");
        try {
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("receive: " + new String(message.getBody()));
        } catch (Exception e) {
            e.printStackTrace();
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
            System.out.println("receiver fail");
        }
    }
```