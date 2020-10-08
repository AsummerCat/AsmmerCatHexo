---
title: kafka笔记之Spring整合kafka-四
date: 2020-10-09 04:12:00
tags: [kafka笔记]
---

# 引入jar包
```
<dependency>
   <groupId>org.springframework.kafka</groupId>
   <artifactId>spring-kafka</artifactId>
   <version>2.1.7.RELEASE</version>
</dependency>
```

<!--more-->


# 生产者配置
## kafka 配置文件
`application-server-kafka.xml`
```
<!--配置文件-->
<bean id="prooducerProperties" class="java.util.HashMap">
   <constructor-arg>
        <map>
           <entry></entry> <!--该部分设置为Config参数 -->
        </map>
   </constructor-arg>
</bean>

<!--生产者工厂-->
<bean id="producerFactory" class="org.springframework.kafka.core.DefaultKafkaProducerFactory">
<!--加载配置文件-->
   <constructor-arg ref="prooducerProperties"></constructor-arg>
</bean>

<!--kafka模板-->
<bean id="kafkaTemplate" class="org.springframework.kafka.core.KafkaTemplate">
    <!--加载工厂-->
   <constructor-arg ref="producerFactory"></constructor-arg>
    <constructor-arg name="autoFlush" value="true"/>
</bean>

```

## 使用
```
@AutoWired
KafkaTemplate kafkaTemplate;


public void send(){
    //发送消息
    kafkaTemplate.send("topic","data");
    
    //
    
}

```

# 消费者配置
## 配置文件
`application-client-kafka.xml`
大体内容相似 ,主要就是 工厂类的实现和
```
<!--配置文件-->
<bean id="consumerProperties" class="java.util.HashMap">
   <constructor-arg>
        <map>
           <entry></entry> <!--该部分设置为Config参数 -->
        </map>
   </constructor-arg>
</bean>

<!--消费者工厂-->
<bean id="consumerFactory" class="org.springframework.kafka.core.DefaultKafkaConsumerFactory">
<!--加载配置文件-->
   <constructor-arg ref="consumerProperties"></constructor-arg>
</bean>


<!--kafka容器配置-->
<bean id="containerProperties" class="org.springframework.kafka.listener.config.ContainerProperties"> 
   <constructor-arg name="topics" value="test"/>
   <!--注册监听器-->
   <property name="messageListener" ref="registryListener"/>
</bean>


<!--kafka消息容器配置-->
<bean id="messageListenerContainer" class="org.springframework.kafka.listener.KafkaMessageListenerContainer" init-method="doStart"> 
   <!--注入消费者工厂-->
   <constructor-arg ref="consumerFactory"/>
   <!--注入容器配置-->
   <constructor-arg ref="containerProperties"/>
</bean>

```

## 配置监听器
订阅内容
```
@Service
public class RegistryListener implements MessageListener<Integer,String>{
    
    
 @Override
 public void onMessage(ConsumerRecord<Integer,String> integerStringConsumerRecord){
     System.out.println("收到消息");
     System.out.println(integerStringConsumerRecord.value());
 }
}

```