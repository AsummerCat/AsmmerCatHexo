---
title: rabbitMq投递queue的参数含义
date: 2019-01-17 13:55:51
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

![参数](/img/2019-1-15/Arguments.png)

参数解释

* Message TTL(x-message-ttl)：设置队列中的所有消息的生存周期(统一为整个队列的所有消息设置生命周期), 也可以在发布消息的时候单独为某个消息指定剩余生存时间,单位毫秒, 类似于redis中的ttl，生存时间到了，消息会被从队里中删除，注意是消息被删除，而不是队列被删除， 特性Features=TTL, 单独为某条消息设置过期时间AMQP.BasicProperties.Builder properties = new AMQP.BasicProperties().builder().expiration(“6000”); 
* Auto Expire(x-expires): 当队列在指定的时间没有被访问(consume, basicGet, queueDeclare…)就会被删除,Features=Exp
* Max Length(x-max-length): 限定队列的消息的最大值长度，超过指定长度将会把最早的几条删除掉， 类似于mongodb中的固定集合，例如保存最新的100条消息, Feature=Lim
* Max Length Bytes(x-max-length-bytes): 限定队列最大占用的空间大小， 一般受限于内存、磁盘的大小, Features=Lim B
* Dead letter exchange(x-dead-letter-exchange)： 当队列消息长度大于最大长度、或者过期的等，将从队列中删除的消息推送到指定的交换机中去而不是丢弃掉,Features=DLX
* Dead letter routing key(x-dead-letter-routing-key)：将删除的消息推送到指定交换机的指定路由键的队列中去, Feature=DLK
* Maximum priority(x-max-priority)：优先级队列，声明队列时先定义最大优先级值(定义最大值一般不要太大)，在发布消息的时候指定该消息的优先级， 优先级更高（数值更大的）的消息先被消费,
* Lazy mode(x-queue-mode=lazy)： Lazy Queues: 先将消息保存到磁盘上，不放在内存中，当消费者开始消费的时候才加载到内存中



## Message TTL(x-message-ttl)  消息过期

数字类型，标志时间，以毫秒为单位

标志队列中的消息存活时间，也就是说队列中的消息超过了制定时间会被删除

```
 @Bean
    public Queue argumentsQueue() {
        Map<String,Object> arguments=new HashMap<>();
        //设置消息过期时间
        arguments.put("x-message-ttl",5000);
        return new Queue("ArgumentsQueue",true,false,false,arguments);
    }
```

到期后 这条消息自动被删除



##Auto Expire(x-expires)  队列无访问过期

数字类型，标志时间，以毫秒为单位

队列自身的空闲存活时间，当前的queue在指定的时间内，没有consumer、basic.get也就是未被访问，就会被删除。

```
@Bean
    public Queue argumentsQueue() {
        Map<String,Object> arguments=new HashMap<>();
        //队列无访问指定时间过期
        arguments.put("x-expires",10000);
        return new Queue("ArgumentsQueue",false,false,false,arguments);
    }
```

队列直接被删除了 如果再次访问 会发送失败 

如果如果设置了publisher-returns: true 

ReturnCallback会显示 应答码：312 原因：NO_ROUTE



## Max Length(x-max-length) 队列最大长度与x-max-length-bytes最大占用空间

数字

最大长度和最大占用空间，设置了最大长度的队列，在超过了最大长度后进行插入会删除之前插入的消息为本次的留出空间,相应的最大占用大小也是这个道理，当超过了这个大小的时候，会删除之前插入的消息为本次的留出空间。

```
 @Bean
    public Queue argumentsQueue() {
        Map<String,Object> arguments=new HashMap<>();
        //队列最大长度
        arguments.put("x-max-length",100);
        //队列最大占用空间
        arguments.put("x-max-length-bytes",10000);
        return new Queue("ArgumentsQueue",false,false,false,arguments);
    }

```

## Dead letter exchange(x-dead-letter-exchange) 超时超限制后 推送给其他交换机

 当队列消息长度大于最大长度、或者过期的等，将从队列中删除的消息推送到指定的交换机中去而不是丢弃掉

字符串

再根据routingkey推到queue中

```
 @Bean
    public Queue argumentsQueue() {
        Map<String,Object> arguments=new HashMap<>();
    arguments.put("x-dead-letter-exchange","timeoutExchange");
        return new Queue("ArgumentsQueue",false,false,false,arguments);
    }
```

消息因为超时或超过限制在队列里消失，这样我们就丢失了一些消息，也许里面就有一些是我们做需要获知的。而rabbitmq的死信功能则为我们带来了解决方案。设置了dead letter exchange与dead letter（要么都设定，要么都不设定）那些因为超时或超出限制而被删除的消息会被推动到我们设置的exchange中，再根据routingkey推到queue中



##  Dead letter routing key(x-dead-letter-routing-key) 删除后 推送给其他交换机

将删除的消息推送到指定交换机的指定路由键的队列中去

字符串

再根据routingkey推到queue中

```
 @Bean
    public Queue argumentsQueue() {
        Map<String,Object> arguments=new HashMap<>();
        arguments.put("x-dead-letter-routing-key","dieExchange");
        return new Queue("ArgumentsQueue",false,false,false,arguments);
    }
```



## Maximum priority(x-max-priority) 优先级队列

优先级队列，声明队列时先定义最大优先级值(定义最大值一般不要太大)，在发布消息的时候指定该消息的优先级， 优先级更高（数值更大的）的消息先被消费,

数字

队列所支持的优先级别，列如设置为5，表示队列支持0到5六个优先级别，5最高，0最低，当然这需要生产者在发送消息时指定消息的优先级别，消息按照优先级别从高到低的顺序分发给消费者

```
 @Bean
    public Queue dieQueue() {
        Map<String, Object> arguments = new HashMap<>();
        arguments.put("x-max-priority",1);
        return new Queue("dieQueue",true,false,false,arguments);
    }
```



## alternate-exchange  单消息不能被路由的时候 会被丢到AE中 继续路由 

下面简称AE，当一个消息不能被route的时候，如果exchange设定了AE，则消息会被投递到AE。如果存在AE链，则会按此继续投递，直到消息被route或AE链结束或遇到已经尝试route过消息的AE。

```
@Bean
    public Queue argumentsQueue() {
        Map<String, Object> arguments = new HashMap<>();
        //路由失败 备用队列
        arguments.put("alternate-exchange","alternateExchange");

        return new Queue("ArgumentsQueue", false, false, false, arguments);
    }
```





