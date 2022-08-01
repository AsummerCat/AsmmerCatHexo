---
title: SpringBoot整合rabbitMq接收消息的各个参数含义
date: 2019-01-17 11:15:01
tags: [SpringBoot,RabbitMQ]
---

## new queue() 基本队列

```
 public Queue(String name, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) {
        Assert.notNull(name, "'name' cannot be null");
        this.name = name;
        this.actualName = StringUtils.hasText(name) ? name : Base64UrlNamingStrategy.DEFAULT.generateName() + "_awaiting_declaration";
        this.durable = durable;
        this.exclusive = exclusive;
        this.autoDelete = autoDelete;
        this.arguments = (Map)(arguments != null ? arguments : new HashMap());
    }
```

```
name: 当前队列的名称
durable: 是否持久化到硬盘 若要使队列中消息不丢失，同时也需要将消息声明为持久化 默认:true
exclusive:是否声明该队列是否为连接独占，若为独占，连接关闭后队列即被删除 默认:false
Auto-delete：若没有消费者订阅该队列，队列将被删除 默认:false
Arguments：可选map类型参数，可以指定队列长度，消息生存时间，镜相设置等 
```

<!--more-->

Arguments参数详情 后一篇文章介绍**

![参数](/img/2019-1-15/Arguments.png)

# 消费端参数



## chanel.basicQos() 控制消费者消费大小

在消费者上创建

这个参数表示 不要同时给一个消费者推送多于N个消息，即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack

```
public void basicQos(int prefetchSize, int prefetchCount, boolean global) throws IOException {
    if (global) {
        this.prefetchCountGlobal = prefetchCount;
    } else {
        this.prefetchCountConsumer = prefetchCount;
    }
```

```
prefetchSize：预读取的消息内容大小上限(包含)，可以简单理解为消息有效载荷字节数组的最大长度限制，0表示无上限。
prefetchCount：预读取的消息数量上限，0表示无上限。
会告诉RabbitMQ不要同时给一个消费者推送多于N个消息，即一旦有N个消息还没有ack，则该consumer将block掉，直到有消息ack
global：false表示prefetchCount单独应用于信道上的每个新消费者，true表示prefetchCount在同一个信道上的消费者共享。
```

案例:

```
案例:
 @RabbitListener(queues = RabbitConfig.BASIC_QUEUE)
    public void process(Message message, Channel channel) throws IOException {
        // 采用手动应答模式, 手动确认应答更为安全稳定
        System.out.println("客户端接收到请求");
        try {
        //每次只接受一条投递的消息
            channel.basicQos(1);
            //告诉服务器收到这条消息 已经被我消费了 可以在队列删掉 这样以后就不会再发了 否则消息服务器以为这条消息没处理掉 后续还会在发
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





## **channel.basicAck()** 控制消息回复 

```
    /**
     * Acknowledge one or several received
     * messages. Supply the deliveryTag from the {@link com.rabbitmq.client.AMQP.Basic.GetOk}
     * or {@link com.rabbitmq.client.AMQP.Basic.Deliver} method
     * containing the received message being acknowledged.
     * @see com.rabbitmq.client.AMQP.Basic.Ack
     * @param deliveryTag the tag from the received {@link com.rabbitmq.client.AMQP.Basic.GetOk} or {@link com.rabbitmq.client.AMQP.Basic.Deliver}
     * @param multiple true to acknowledge all messages up to and
     * including the supplied delivery tag; false to acknowledge just
     * the supplied delivery tag.
     * @throws java.io.IOException if an error is encountered
     */
    void basicAck(long deliveryTag, boolean multiple) throws IOException;
```

```
deliveryTag:该消息的index 
multiple：是否批量.  true:将一次性ack所有小于deliveryTag的消息。 false:只消费当前deliveryTag消息
```



## **channel.basicNack** 拒绝消息(可以批量) 是否重发或者拒绝

```
 /**
     * Reject one or several received messages.
     *
     * Supply the <code>deliveryTag</code> from the {@link com.rabbitmq.client.AMQP.Basic.GetOk}
     * or {@link com.rabbitmq.client.AMQP.Basic.GetOk} method containing the message to be rejected.
     * @see com.rabbitmq.client.AMQP.Basic.Nack
     * @param deliveryTag the tag from the received {@link com.rabbitmq.client.AMQP.Basic.GetOk} or {@link com.rabbitmq.client.AMQP.Basic.Deliver}
     * @param multiple true to reject all messages up to and including
     * the supplied delivery tag; false to reject just the supplied
     * delivery tag.
     * @param requeue true if the rejected message(s) should be requeued rather
     * than discarded/dead-lettered
     * @throws java.io.IOException if an error is encountered
     */
    void basicNack(long deliveryTag, boolean multiple, boolean requeue)

```

```
deliveryTag:该消息的index
multiple：是否批量.true:将一次性拒绝所有小于deliveryTag的消息。 
requeue：被拒绝的是否重新入队列 
```

案例:

```
//重新放入队列中
            //channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
            //丢弃这条消息
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
```



## **channel.basicReject()** 拒绝当前消息 

```
/**
     * Reject a message. Supply the deliveryTag from the {@link com.rabbitmq.client.AMQP.Basic.GetOk}
     * or {@link com.rabbitmq.client.AMQP.Basic.Deliver} method
     * containing the received message being rejected.
     * @see com.rabbitmq.client.AMQP.Basic.Reject
     * @param deliveryTag the tag from the received {@link com.rabbitmq.client.AMQP.Basic.GetOk} or {@link com.rabbitmq.client.AMQP.Basic.Deliver}
     * @param requeue true if the rejected message should be requeued rather than discarded/dead-lettered
     * @throws java.io.IOException if an error is encountered
     */
    void basicReject(long deliveryTag, boolean requeue) throws IOException;
```

```
deliveryTag:该消息的index
requeue：被拒绝的是否重新入队列

channel.basicNack 与 channel.basicReject 的区别在于basicNack可以拒绝多条消息，而basicReject一次只能拒绝一条消息
```

案例:

```
            channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
```

