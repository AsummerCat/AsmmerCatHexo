---
title: SpringBoot整合ActiveMQ
date: 2018-10-17 15:23:14
tags: [SpringBoot,消息队列,ActiveMQ]
---

#JMS四个重要部分
```
消息主题(Topic)，简单来说就是群发 
消息队列(MQ) ，存放或者实现JMS的功能，需要用到队列，有人放入消息到外卖队列，有人从外卖队列读取消息，就是一个消息队列的模型。
发送者(Sender)，通过什么事或者想做什么事，就发个信息，例如点个外卖呗。
接收者(Receiver)，然后外卖员就收到信息，办事。
！
```
<!--more-->

#ActiveMQ下载安装
```
 　  下载地址: http://activemq.apache.org/download.html

　　安装过程比较简单, 在centos中, 解压出来, 就算是安装好了

　　运行方法: 
　　解压目录下 ./bin/activemq start
　　默认端口8161
　　默认账号密码: admin admin
　　# 分为两个端口 其中 8161是本地后台的   61616是远程连接到active的端口号

```

#导入pom.xml

```
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-activemq</artifactId>
      </dependency>
如果使用pool的话, 就需要在pom中加入以下依赖:
<dependency>
     <groupId>org.apache.activemq</groupId>
     <artifactId>activemq-pool</artifactId>
     <version>5.14.5</version>
</dependency>

```
---

# Activemq配置详情

```
spring.activemq.broker-url= true   指定ActiveMQ broker的URL，默认自动生成.

spring.activemq.in-memory= true    是否是内存模式，默认为true.

spring.activemq.password=         指定broker的密码.

spring.activemq.pooled=    是否创建PooledConnectionFactory，而非ConnectionFactory，默认false

spring.activemq.user=     指定broker的用户.

spring.activemq.packages.trust-all=false    信任所有的包
如果传输的对象是Obeject 这里必须加上这句或者指定信任的包  否则会导致对象序列化失败 出现classnotfound异常  详细： http://activemq.apache.org/objectmessage.html

spring.activemq.packages.trusted=   指定信任的包,当有信任所有的包为true是无效的

spring.activemq.pool.configuration.*=    暴露pool配置

spring.activemq.pool.enabled=false     是否使用PooledConnectionFactory

spring.activemq.pool.expiry-timeout=0    连接超时时间

#spring.activemq.pool.idle-timeout=30000     空闲时间

spring.activemq.pool.max-connections=1    最大连接数

spring.jms.jndi-name    指定Connection factory JNDI 名称.

spring.jms.listener.acknowledge-mode   指定ack模式，默认自动ack.

spring.jms.listener.auto-startup     是否启动时自动启动jms，默认为: true

spring.jms.listener.concurrency     指定最小的并发消费者数量.

spring.jms.listener.max-concurrency      指定最大的并发消费者数量.

spring.jms.pub-sub-domain    是否使用默认的destination type来支持 publish/subscribe，默认: false
```

---

#在SpringBoot的配置文件中写入

 ```
#activemq整合
# 分为两个端口 其中 8161是本地后台的   61616是远程连接到active的端口号
spring.activemq.broker-url=tcp://localhost:61616/
#集群配置
#spring.activemq.broker-url=failover:(tcp://172.18.1.188:61616,tcp://172.18.1.18:61616)
#activeMQ用户名，根据实际情况配置
spring.activemq.user=admin
#activeMQ密码，根据实际情况配置
spring.activemq.password=admin
#是否启用内存模式（也就是不安装MQ，项目启动时同时也启动一个MQ实例）
spring.activemq.in-memory=true
#是否替换默认的connectionFactory,是否创建PooledConnectionFactory，默认false
spring.activemq.pool.enabled=false
#最大连接数
spring.activemq.pool.maxConnections=2
#空闲时间
spring.activemq.pool.idleTimeout=30
#信任所有的包
spring.activemq.packages.trust-all=true


 ```
---

#在入口类加入 注解

开启jms注解
`@EnableJms  //这个是jms 注解`

**需要注意的是:<font color="red">默认是不接收top请求的 需要做如下操作才可以</font>** 

在入口类中加入

```
/**
	 * 开启接收top请求
	 * 将pubSubDomain设置成了true，表示该Listener负责处理Topic
	 * 默认是 Queue
	 */
	@Bean
	public JmsListenerContainerFactory<?> jmsListenerContainerTopic(ConnectionFactory activeMQConnectionFactory) {
		DefaultJmsListenerContainerFactory bean = new DefaultJmsListenerContainerFactory();
		bean.setPubSubDomain(true);
		bean.setConnectionFactory(activeMQConnectionFactory);
		return bean;
	}
```

---

#例子
##创建生产者

```
package com.linjingc.demo.test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsMessagingTemplate;
import org.springframework.stereotype.Service;

import javax.jms.Destination;

/**
 * 创建生产者
 */
@Service("producer")
public class Producer {
    @Autowired // 也可以注入JmsTemplate，JmsMessagingTemplate对JmsTemplate进行了封装
    private JmsMessagingTemplate jmsTemplate;

    // 发送消息，destination是发送到的队列，message是待发送的消息
    public void sendMessage(Destination destination, final String message) {
        jmsTemplate.convertAndSend(destination, message);
    }
}


```

##创建消费者 (可以监听消息)
监听 `mytest.queue` 队列

```
package com.linjingc.demo.test;

import lombok.extern.slf4j.Slf4j;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Component;

/**
 * 创建消费者
 */
@Component
@Slf4j
public class Consumer {
    // 使用JmsListener配置消费者监听的队列，其中text是接收到的消息
    @JmsListener(destination = "mytest.queue")
    @SendTo({"out.queue"})
    public String receiveQueue(String text) {
        System.out.println("Consumer收到的报文为:" + text);
        return "out.queue发送了";
    }
}

```

## 生产者发送请求

```
 @RequestMapping("/hello")
    public String hello(){
        Destination destination = new ActiveMQQueue("mytest.queue");
        producer.sendMessage(destination,"看不到我");
        return "发送一个请求";
    }
```

---


#生产者须知
需要注入 `JmsMessagingTemplate` 实现发送消息

```
 @Autowired    // 也可以注入JmsTemplate，JmsMessagingTemplate对JmsTemplate进行了封装
 private JmsMessagingTemplate jmsTemplate;
```
或者 
当然你也可以直接在直接注入这个方法实现 `convertAndSend(Destination destination, final String message)`这个方法  
前一个参数代表 发送类型 后一个参数 代表 消息

```
Destination destination = new ActiveMQTopic("mytest.topic");  //主题
Destination destination = new ActiveMQQueue("mytest.queue");  //队列
Destination destination = new ActiveMQTempQueue("mytest.queue1"); //临时队列  
Destination destination = new ActiveMQTempTopic("mytest.queue1"); //临时主题


```

---

# 消费者须知
这里可以 使用注解来监听队列消息  
` @JmsListener(destination = "mytest.topic")`  

之前提到过默认是不接收`Top`请求的
所以我们之前在入口类中创建了一个bean 来实现接收top请求

所以这里接收top请求的注解需要这么写:  
在原来注解基础上加入 `containerFactory ="jmsListenerContainerTopic"`
也就是入口处的bean

```
 @JmsListener(destination = "mytest.topic", containerFactory = "jmsListenerContainerTopic")  
     //这个注解是监听信息 主题  名称

```

```
@SendTo({"out.queue"})

这个可以放在消费者这里 将返回值发送出去  注意 void 就不会发送了

```

---

# 注解:

##  `@EnableJms` 开启JMS注解  

##  ` @JmsListener(destination = "mytest.topic")` 监听队列

##  `@SendTo({"out.queue"})`  发送消息


