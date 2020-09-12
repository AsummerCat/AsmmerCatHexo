---
title: rabbitMQ笔记优化方面的问题(四)
date: 2020-09-12 14:37:20
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记优化方面的问题(四)

#  如何确保消息一定发送到Rabbitmq了?
在正常情况下 是没问题的， 但是实际开发中 我们往往要考虑一些非正常的情况，我们从
消息的发送开始：  
默认情况下，我们不知道我们的消息到底有没有发送到rabbitmq当中， 这肯定是不可取的， 假设我们是一个电商
项目的话 用户下了订单 订单发送消息给库存 结果这个消息没发送到rabbitmq当中 但是订单还是下了，这时候 因
为没有消息 库存不会去减少库存， 这种问题是非常严重的， 所以 接下来就讲一种解决方案：  
<!--more-->
### <font color="red">失败回调</font>
```
失败回调， 顾名思义 就是消息发送失败的时候会调用我们事先准备好的回调函数，
并且把失败的消息 和失败原因等 返回过来。
具体操作：
注意 使用失败回调也需要开启发送方确认模式 开启方式在下文
```
更改RabbitmqTemplate:
```
@Bean
    public RabbitTemplate rabbitTemplate() {
        Logger log = LoggerFactory.getLogger(RabbitTemplate.class);
        RabbitTemplate rabbitTemp = new RabbitTemplate(connectionFactory);
        // 消息发送失败返回到队列中, yml需要配置 publisher-returns: true
        rabbitTemp.setMandatory(true);

        //交换机发送消息给队列失败回调的处理
        //指定失败回调接口的实现类
        rabbitTemp.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            String correlationId = message.getMessageProperties().toString();
            log.info("消息：{} 发送失败, 应答码：{} 原因：{} 交换机: {}  路由键: {}", correlationId, replyCode, replyText, exchange, routingKey);
        });
        return rabbitTemp;
}        
        
```
### 需要注意一点 虽然支持事务但是性能很差
事物的确能解决这个问题， 而且 恰巧rabbitmq刚好也支持事物，  
但是！   
事物非常影响rabbitmq的性能 有
多严重？  
据我所查到的资料开启rabbitmq事物的话
对性能的影响超过100倍之多 也就是说 开启事物后处理一条消息的时间 不开事物能处理100条（姑且这样认为
吧）   
那么 这样是非常不合理的， 因为消息中间件的性能其实非常关键的（参考双11） 如果这样子做的话 虽然
能确保消息100%投递成功 但是代价太大了！
### 发送方确认模式 
这种方式 对性能的影响非常小 而且也能确定消息是否
发送成功    

而且 <font color="red">发送方确认模式一般也会和失败回调一起使用</font> 这样 就能确保消息100%投递了  
### 完整代码:
```
/**
 * RabbitMQ配置类
 */
@Configuration
public class RabbitConfig {
    @Autowired
    private ConnectionFactory connectionFactory;
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

        // 交换机发送消息给队列失败返回, yml需要配置 publisher-returns: true
        rabbitTemp.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            String correlationId = message.getMessageProperties().toString();
            log.info("消息：{} 发送失败, 应答码：{} 原因：{} 交换机: {}  路由键: {}", correlationId, replyCode, replyText, exchange, routingKey);
        });

        // 生产者发送消息mq后确认, yml需要配置 publisher-confirms: true
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

---


## 发送消息的时候除了消息本体还可以附带一个业务id
写法如下:
```
//业务id
CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString()); 
rabbitTemplate.convertAndSend("directExchange", "direct.key123123", "hello",correlationData);
```