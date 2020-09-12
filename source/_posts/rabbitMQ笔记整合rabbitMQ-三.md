---
title: rabbitMQ笔记整合rabbitMQ(三)
date: 2020-09-12 14:36:39
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记整合rabbitMQ(三)

## 导入pom
```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
		
还要加入一个json的依赖		

```
<!--more-->

## 修改application.yml
模板:
```
spring: 
  rabbitmq: 
    host: 
    port: 
    username: 
    password: 
    virtual-host:
```
实例:
```
spring:
  application:
    name:  boot-rabbitMq
  rabbitmq:
    host: 112.74.43.136
    port: 5672
    username: cat
    password: cat
    publisher-confirms: true #  生产者消息发送到交换机确认机制,是否确认回调
    publisher-returns: true  # 消费者消费消息后返回确认 
    ##开启ack
    listener:
      direct:
      ## 采取手动应答
        acknowledge-mode: manual
      simple:
        acknowledge-mode: manual
```

## 创建一个RabbitMQ的配置类
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

## 因为传输的是json 我们可以定义一个转换器来避免手动转换json
```
在上面RabbitTemplate中加入转换器

rabbitTemp.setMessageConverter(new MessageConverter() {
           //发送消息要转换的  生产者重写
            @Override
            public Message toMessage(Object o, MessageProperties messageProperties) throws MessageConversionException {
               //指定输出格式
                messageProperties.setContentType("text/xml");
                messageProperties.setContentEncoding("UTF-8");
                Message message=new Message(JSON.toJSONBytes(o),messageProperties);
                System.out.println("调用了消息解析器");
                return null;
            }

            //接收消息要转换的  消费者者重写
            @Override
            public Object fromMessage(Message message) throws MessageConversionException {
              //使用json解析
                System.out.println(new String(message.getBody(),"UTF-8"));
                return null;
            }
        });
```

## 代码写法
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