---
title: Springboot整合Mqtt
date: 2022-10-19 13:53:09
tags: [MQTT,SpringBoot]
---
# Springboot整合Mqtt

## 测试demo

https://github.com/AsummerCat/MqttTest.git

## 引入相关依赖
```
       <!--MQTT依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-integration</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-mqtt</artifactId>
        </dependency>


        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.83</version>
        </dependency>
```
<!--more-->

## 编写MQTT配置文件
注意 每个clientid 只能存在一个 不然会导致连接掉线 唯一标识
```
spring:
  mqtt:
    enable: true
    url: tcp://broker.emqx.io
    username:
    password:
    consumerclientid: clientid12342312212_c
    # 自定义编码消费者id
    userconsumerclientid: clientid12342312212_d
    # 生产者id
    producerclientid: clientid12342312212_p
    # 自定义编码生产者id
    userproducerclientid: clientid12342312212_g
    timeout: 5000
    # 心跳时间
    keepalive: 2
    # mqtt-topic
    producertopic: bstes,user
    consumertopic: bstes
```

## 编写MQTT配置类
连接配置 ,生产者配置,消费者配置, 自定义编码器配置
```
package com.linjingc.mqttandprotocol.config;

import com.linjingc.mqttandprotocol.converter.UserConverter;
import com.linjingc.mqttandprotocol.mqtt.MqttConsumer;
import com.linjingc.mqttandprotocol.mqtt.UserMqttConsumer;
import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.IntegrationComponentScan;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.core.MessageProducer;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.inbound.MqttPahoMessageDrivenChannelAdapter;
import org.springframework.integration.mqtt.outbound.MqttPahoMessageHandler;
import org.springframework.integration.mqtt.support.DefaultPahoMessageConverter;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;


@Configuration
@IntegrationComponentScan
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
@Slf4j
public class MqttConfig {
    @Value("${spring.mqtt.username}")
    private String username;

    @Value("${spring.mqtt.password}")
    private String password;

    @Value("${spring.mqtt.url}")
    private String hostUrl;

    @Value("${spring.mqtt.producerclientid}")
    private String producerClientId;
    @Value("${spring.mqtt.userproducerclientid}")
    private String userProducerClientId;

    @Value("${spring.mqtt.producertopic}")
    private String producerTopic;

    //生产者和消费者是单独连接服务器会使用到一个clientid（客户端id），如果是同一个clientid的话会出现Lost connection: 已断开连接; retrying...
    @Value("${spring.mqtt.consumerclientid}")
    private String consumerClientId;
    @Value("${spring.mqtt.userconsumerclientid}")
    private String userConsumerclientid;

    @Value("${spring.mqtt.consumertopic}")
    private String[] consumerTopic;

    @Value("${spring.mqtt.timeout}")
    private int timeout;   //连接超时

    @Value("${spring.mqtt.keepalive}")
    private int keepalive;  //连接超时


    @Autowired
    /**
     * 自定义编码器
     */
    private UserConverter userConverter;


    /**
     * MQTT客户端
     *
     * @return {@link org.springframework.integration.mqtt.core.MqttPahoClientFactory}
     */
    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        //MQTT连接器选项
        MqttConnectOptions mqttConnectOptions = new MqttConnectOptions();
        mqttConnectOptions.setUserName(username);
        mqttConnectOptions.setPassword(password.toCharArray());
        mqttConnectOptions.setServerURIs(new String[]{hostUrl});
        mqttConnectOptions.setKeepAliveInterval(keepalive);
        factory.setConnectionOptions(mqttConnectOptions);
        return factory;
    }

    /*******************************生产者*******************************************/

    /**
     * MQTT信息通道（生产者）
     *
     * @return {@link MessageChannel}
     */
    @Bean(name = "mqttOutboundChannel")
    public MessageChannel mqttOutboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息处理器（生产者）
     *
     * @return
     */
    @Bean
    //出站通道名（生产者）
    @ServiceActivator(inputChannel = "mqttOutboundChannel")
    public MessageHandler mqttOutbound() {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(producerClientId, mqttClientFactory());
        //如果设置成true，发送消息时将不会阻塞。
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(producerTopic);
        return messageHandler;
    }

    /*******************************消费者*******************************************/

    /**
     * MQTT信息通道（消费者）
     *
     * @return {@link MessageChannel}
     */
    @Bean(name = "mqttInboundChannel")
    public MessageChannel mqttInboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息订阅绑定（消费者）
     *
     * @return {@link org.springframework.integration.core.MessageProducer}
     */
    @Bean
    public MessageProducer inbound() {
        // 可以同时消费（订阅）多个Topic
        MqttPahoMessageDrivenChannelAdapter adapter = new MqttPahoMessageDrivenChannelAdapter(consumerClientId, mqttClientFactory(), consumerTopic);
        adapter.setCompletionTimeout(timeout);
        //默认编码器
        adapter.setConverter(new DefaultPahoMessageConverter());
//        自定义编码器
//        adapter.setConverter(userConverter);

        //qos是mqtt 对消息处理的几种机制分为0,1,2
        // 其中0表示的是订阅者没收到消息不会再次发送,消息会丢失,
        // 1表示的是会尝试重试,一直到接收到消息,但这种情况可能导致订阅者收到多次重复消息,
        // 2相比多了一次去重的动作,确保订阅者收到的消息有一次
        // 当然,这三种模式下的性能肯定也不一样,qos=0是最好的,2是最差的
        adapter.setQos(1);
        // 设置订阅通道
        adapter.setOutputChannel(mqttInboundChannel());
        return adapter;
    }

    /**
     * MQTT消息处理器（消费者）
     * ServiceActivator注解表明当前方法用于处理MQTT消息，inputChannel参数指定了用于接收消息信息的channel。
     *
     * @return {@link MessageHandler}
     */
    @Bean
    //入站通道名（消费者）
    @ServiceActivator(inputChannel = "mqttInboundChannel")
    public MessageHandler handler() {
        return new MqttConsumer();
    }


    /*******************************自定义编码生产者*******************************************/

    @Bean(name = "userMqttOutboundChannel")
    public MessageChannel userMqttOutboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息处理器（生产者）
     *
     * @return
     */
    @Bean
    //出站通道名（生产者）
    @ServiceActivator(inputChannel = "userMqttOutboundChannel")
    public MessageHandler userMqttOutbound() {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(userProducerClientId, mqttClientFactory());
        //如果设置成true，发送消息时将不会阻塞。
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(producerTopic);
        messageHandler.setConverter(userConverter);
        return messageHandler;
    }

    /*******************************自定义编码消费者*******************************************/

    @Bean(name = "userMqttInboundChannel")
    public MessageChannel userMqttInboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息订阅绑定（消费者）
     *
     * @return {@link org.springframework.integration.core.MessageProducer}
     */
    @Bean
    public MessageProducer userInbound() {
        // 可以同时消费（订阅）多个Topic
        MqttPahoMessageDrivenChannelAdapter adapter = new MqttPahoMessageDrivenChannelAdapter(userConsumerclientid, mqttClientFactory(), "user");
        adapter.setCompletionTimeout(timeout);
        //自定义编码器
        adapter.setConverter(userConverter);
        adapter.setQos(1);
        // 设置订阅通道
        adapter.setOutputChannel(userMqttInboundChannel());
        return adapter;
    }

    @Bean
    //入站通道名（消费者）
    @ServiceActivator(inputChannel = "userMqttInboundChannel")
    public MessageHandler userHandler() {
        return new UserMqttConsumer();
    }
}
 
```

## 消费端接收消息类
```
package com.linjingc.mqttandprotocol.mqtt;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;
import org.springframework.stereotype.Component;

@Component
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
@Slf4j
public class MqttConsumer implements MessageHandler {

    @Override
    public void handleMessage(Message<?> message) throws MessagingException {
        String topic = String.valueOf(message.getHeaders().get(MqttHeaders.RECEIVED_TOPIC));
        String payload = String.valueOf(message.getPayload());
        log.info("接收到 mqtt消息，主题:{} 消息:{}", topic, payload);
    }
}
```
## 自定义消费端编码接收消息类
```
package com.linjingc.mqttandprotocol.mqtt;

import com.alibaba.fastjson.JSON;
import com.linjingc.mqttandprotocol.converter.User;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;
import org.springframework.stereotype.Component;

/**
 * 自定义编码消息
 */
@Component
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
@Slf4j
public class UserMqttConsumer implements MessageHandler {

    @Override
    public void handleMessage(Message<?> message) throws MessagingException {
        String topic = String.valueOf(message.getHeaders().get(MqttHeaders.RECEIVED_TOPIC));
        User user = (User) message.getPayload();
        log.info("User接收到 mqtt消息，主题:{} 消息:{}", topic, JSON.toJSONString(user));
    }
}
```

## 生产者发送消息类
```
package com.linjingc.mqttandprotocol.mqtt;

import com.linjingc.mqttandprotocol.converter.User;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.integration.annotation.MessagingGateway;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

/**
 * 生产者发送消息
 */

//@MessagingGateway是一个用于提供消息网关代理整合的注解，参数defaultRequestChannel指定发送消息绑定的channel。
@MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
@Component
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
public interface MqttProducer {

    /**
     * payload或者data是发送消息的内容
     * topic是消息发送的主题,这里可以自己灵活定义,也可以使用默认的主题,就是配置文件的主题,qos是mqtt 对消息处理的几种机制分为0,1,2 其中0表示的是订阅者没收到消息不会再次发送,消息会丢失,1表示的是会尝试重试,一直到接收到消息,但这种情况可能导致订阅者收到多次重复消息,2相比多了一次去重的动作,确保订阅者收到的消息有一次
     * 当然,这三种模式下的性能肯定也不一样,qos=0是最好的,2是最差的
     */

    void sendToMqtt(String data);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, String payload);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) int qos, String payload);


    //自定义编码数据
    void sendToMqtt(User user);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, User user);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) Integer Qos, User user);
}
```

## 自定义编码生产者发送消息类
```
package com.linjingc.mqttandprotocol.mqtt;

import com.linjingc.mqttandprotocol.converter.User;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.integration.annotation.MessagingGateway;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

/**
 * 生产者发送消息
 */

//@MessagingGateway是一个用于提供消息网关代理整合的注解，参数defaultRequestChannel指定发送消息绑定的channel。
@MessagingGateway(defaultRequestChannel = "userMqttOutboundChannel")
@Component
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
public interface UserMqttProducer {
    //自定义编码数据
    void sendToMqtt(User user);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, User user);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) Integer Qos, User user);
}
```

## 自定义编码消息转换类
```
package com.linjingc.mqttandprotocol.converter;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.integration.mqtt.support.MqttMessageConverter;
import org.springframework.integration.support.AbstractIntegrationMessageBuilder;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

@Component
@Slf4j
public class UserConverter implements MqttMessageConverter {
    private int defaultQos = 0;
    private boolean defaultRetain = false;
    ObjectMapper om = new ObjectMapper();

    //入站消息解码
    @Override
    public AbstractIntegrationMessageBuilder<User> toMessageBuilder(String topic, MqttMessage mqttMessage) {
        User protocol = null;
        try {
            protocol = om.readValue(mqttMessage.getPayload(), User.class);
        } catch (IOException e) {
            if (e instanceof JsonProcessingException) {
                System.out.println();
                log.error("Converter only support json string");
            }
        }
        assert protocol != null;
        MessageBuilder<User> messageBuilder = MessageBuilder
                .withPayload(protocol);
        //使用withPayload初始化的消息缺少头信息，将原消息头信息填充进去
        messageBuilder.setHeader(MqttHeaders.ID, mqttMessage.getId())
                .setHeader(MqttHeaders.RECEIVED_QOS, mqttMessage.getQos())
                .setHeader(MqttHeaders.DUPLICATE, mqttMessage.isDuplicate())
                .setHeader(MqttHeaders.RECEIVED_RETAINED, mqttMessage.isRetained());
        if (topic != null) {
            messageBuilder.setHeader(MqttHeaders.TOPIC, topic);
        }
        return messageBuilder;
    }

    //出站消息编码
    @Override
    public Object fromMessage(Message<?> message, Class<?> targetClass) {
        MqttMessage mqttMessage = new MqttMessage();
        String msg = null;
        try {
            msg = om.writeValueAsString(message.getPayload());
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        assert msg != null;
        mqttMessage.setPayload(msg.getBytes(StandardCharsets.UTF_8));
        //这里的 mqtt_qos ,和 mqtt_retained 由 MqttHeaders 此类得出不可以随便取，如需其他属性自行查找
        Integer qos = (Integer) message.getHeaders().get("mqtt_qos");
        mqttMessage.setQos(qos == null ? defaultQos : qos);
        Boolean retained = (Boolean) message.getHeaders().get("mqtt_retained");
        mqttMessage.setRetained(retained == null ? defaultRetain : retained);
        return mqttMessage;
    }

    //此方法直接拿默认编码器的来用的，照抄即可
    @Override
    public Message<?> toMessage(Object payload, MessageHeaders headers) {
        Assert.isInstanceOf(MqttMessage.class, payload,
                () -> "This converter can only convert an 'MqttMessage'; received: " + payload.getClass().getName());
        return this.toMessage(null, (MqttMessage) payload);
    }

    public void setDefaultQos(int defaultQos) {
        this.defaultQos = defaultQos;
    }

    public void setDefaultRetain(boolean defaultRetain) {
        this.defaultRetain = defaultRetain;
    }
}

```
## 自定义编码实体类
```
package com.linjingc.mqttandprotocol.converter;

//协议对象
public class User {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## 测试类
```
package com.linjingc.mqttandprotocol.controller;

import com.alibaba.fastjson.JSON;
import com.linjingc.mqttandprotocol.mqtt.MqttProducer;
import com.linjingc.mqttandprotocol.converter.User;
import com.linjingc.mqttandprotocol.mqtt.UserMqttProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MqttController {

    @Autowired
    private MqttProducer mqttProducer;
    @Autowired
    private UserMqttProducer userMqttProducer;

    @RequestMapping("/send/{topic}/{message}")
    public String send(@PathVariable String topic, @PathVariable String message) {
        mqttProducer.sendToMqtt(topic, message);
        return "send message : " + message;
    }


    @RequestMapping("/user")
    public String send1() {
        User user = new User();
        user.setUsername("小明");
        user.setPassword("密码");
        userMqttProducer.sendToMqtt("user", user);
        return "send message : " + JSON.toJSONString(user);
    }


}

```