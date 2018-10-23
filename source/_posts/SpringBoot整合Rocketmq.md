---
title: SpringBoot整合Rocketmq
date: 2018-10-18 16:53:51
tags: [RocketMQ,消息队列,SpringBoot]
---

# 导入相关pom.xml

```
<!--客户端-->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.3.1</version>
</dependency>

```

<!--more--> 

# 编写Producer
RocketMq有3中消息类型  

 1. 普通消费

 2. 顺序消费

 3. 事务消费

```
package com.linjing.demo;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

/**
 * @author cxc
 * @date 2018/10/18 17:07
 */
public class Producer {

    public static void main(String[] args) throws MQClientException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("producer_demo");
        //指定NameServer地址
        //修改为自己的
        //producer.setNamesrvAddr("192.168.116.115:9876;192.168.116.116:9876");
        producer.setNamesrvAddr("112.74.43.136:9876");

        /**
         * Producer对象在使用之前必须要调用start初始化，初始化一次即可
         * 注意：切记不可以在每次发送消息时，都调用start方法
         */
        producer.start();

        for (int i = 0; i < 997892; i++) {
            try {
                //构建消息
                Message msg = new Message("TopicTest" /* Topic */,
                        "TagA" /* Tag */,
                        ("测试RocketMQ" + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
                );
                //发送同步消息
                SendResult sendResult = producer.send(msg);

                System.out.printf("%s%n", sendResult);
            } catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }
        producer.shutdown();
    }
}


```

# 编写 Consumer

```
package com.linjing.demo.message.ordinary;

import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerOrderly;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;

import java.util.List;

/**
 * @author cxc
 * @date 2018/10/18 17:07
 */
public class OrdinaryConsumer {
    public static void main(String[] args) throws MQClientException {
        /**
         * Consumer Group,非常重要的概念，后续会慢慢补充
         */
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_demo");
//指定NameServer地址，多个地址以 ; 隔开
//        consumer.setNamesrvAddr("112.74.43.136:9876;192.168.116.116:9876"); //修改为自己的
        consumer.setNamesrvAddr("112.74.43.136:9876");

/**
 * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费
 * 如果非第一次启动，那么按照上次消费的位置继续消费
 */
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);


        consumer.subscribe("TopicTest", "TagA || TagC || TagD");

        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
      consumeOrderlyContext.setAutoCommit(true);
                try {
                    for (MessageExt msg : list) {
                        String msgbody = new String(msg.getBody(), "utf-8");
                        System.out.println("  MessageBody: " + msgbody);//输出消息内容
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                    return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT; //稍后再试
                }
                //返回消费状态
                //SUCCESS 消费成功
                //SUSPEND_CURRENT_QUEUE_A_MOMENT 消费失败，暂停当前队列的消费
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });


        consumer.start();

        System.out.printf("Consumer Started.%n");
    }
}


```

**搞定**