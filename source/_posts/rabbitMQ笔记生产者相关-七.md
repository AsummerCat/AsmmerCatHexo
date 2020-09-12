---
title: rabbitMQ笔记生产者相关(七)
date: 2020-09-12 14:38:59
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记生产者相关(七)

# 生产者如何保证消息一定发送成功
需要两步
1. 生产发送到交换机确认发送成功
2. 交换机发送给队列失败需要回调
<!--more-->

### 配置写法 yml
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
    publisher-returns: true  # 交换机发送消息给队列返回确认
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
### 配置写法 配置类
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

        // 生产者发送消息给mq后确认, yml需要配置 publisher-confirms: true
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



# 发送普通消息
```
rabbitTemplate.convertAndSend(RabbitConfig.DIRECT_EXCHANGE, "point", context, correlationData);
```
# 事务消息
性能低下不建议使用
### 配置写法
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
# 特别注意一个事情 死信队列
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

# MQ常用相关配置
```
下面会列出rabbitmq的常用配置：
队列配置：
参数名	配置作用
x-dead-letter-exchange	死信交换机
x-dead-letter-routing-key	死信消息重定向路由键
x-expires	队列在指定毫秒数后被删除
x-ha-policy	创建HA队列
x-ha-nodes	HA队列的分布节点
x-max-length	队列的最大消息数
x-message-ttl	毫秒为单位的消息过期时间，队列级别
x-max-priority	最大优先值为255的队列优先排序功能
消息配置：
参数名	配置作用
content-type	消息体的MIME类型，如application/json
content-encoding	消息的编码类型
message-id	消息的唯一性标识，由应用进行设置
correlation-id	一般用做关联消息的message-id，常用于消息的响应
timestamp	消息的创建时刻，整形，精确到秒
expiration	消息的过期时刻， 字符串，但是呈现格式为整型，精确到秒
delivery-mode	消息的持久化类型，1为非持久化，2为持久化，性能影响巨大
app-id	应用程序的类型和版本号
user-id	标识已登录用户，极少使用
type	消息类型名称，完全由应用决定如何使用该字段
reply-to	构建回复消息的私有响应队列
headers	键/值对表，用户自定义任意的键和值
priority	指定队列中消息的优先级

```