---
title: RocketMQ发送消息的三种方式
date: 2018-10-19 17:20:41
tags: [RocketMQ,消息队列]
---
 
# MQ 发送消息有三种实现方式：
 
 可靠同步发送(`send`)、可靠异步发送(`sendAsync`)、单向(`Oneway`)发送。注意：顺序消息只支持可靠同步发送。
 
 ---
 
 
 
# 发送模式

<!--more-->

## 可靠同步发送
* 原理：同步发送是指消息发送方发出数据后，会在收到接收方发回响应之后才发下一个数据包的通讯方式。
* 场景：此种方式应用场景非常广泛，例如重要通知邮件、报名短信通知、营销短信系统等。 
* 类似推拉的形式  `发送 ->同步返回 ->发送 ->同步返回`

## 可靠异步发送
* 原理：异步发送是指发送方发出数据后，不等接收方发回响应，接着发送下个数据包的通讯方式。 MQ 的异步发送，需要用户实现异步发送回调接口（SendCallback）。消息发送方在发送了一条消息后，不需要等待服务器响应即可返回，进行第二条消息发送。发送方通过回调接口接收服务器响应，并对响应结果进行处理。
* 场景：异步发送一般用于链路耗时较长，对响应时间较为敏感的业务场景，例如用户视频上传后通知启动转码服务，转码完成后通知推送转码结果等。 耗时比较长的 可以不需要同步返回给用户的

## 单向（Oneway）发送
* 原理：单向（Oneway）发送特点为发送方只负责发送消息，不等待服务器回应且没有回调函数触发，即只发送请求不等待应答。 此方式发送消息的过程耗时非常短，一般在微秒级别。
* 场景：适用于某些耗时非常短，但对可靠性要求并不高的场景，例如日志收集。

---

## 三者的特点和主要区别

```
发送方式	发送 TPS	发送结果反馈	可靠性
---------------------------------------
同步发送	  快	       有	        不丢失
异步发送	  快        	有     	不丢失
单向发送	 最快      	无	      可能丢失
```
---

# 例子
## 同步发送

```
package com.linjing.demo.message.ordinary;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.io.UnsupportedEncodingException;
import java.util.Date;

/**
 * @author cxc
 * @date 2018/10/18 17:07
 * rockerMq发送普通消息
 */
public class Producer {

    public static void main(String[] args) throws MQClientException, InterruptedException, UnsupportedEncodingException {
        DefaultMQProducer producer = new DefaultMQProducer("producer_demo");
        //指定NameServer地址
        //修改为自己的
        //多个可以用";"隔开
        //producer.setNamesrvAddr("192.168.116.115:9876;192.168.116.116:9876");
        producer.setNamesrvAddr("112.74.43.136:9876");

        /*
         * Producer对象在使用之前必须要调用start初始化，初始化一次即可
         * 注意：切记不可以在每次发送消息时，都调用start方法
         */
        producer.start();

        for (int i = 0; i <= 100; i++) {

                /*
                构建消息
                参数  topic:   Message 所属的 Topic
                      tags:   可理解为对消息进行再归类，方便 Consumer 指定过滤条件在 MQ 服务器过滤
                      keys:   设置代表消息的业务关键属性，请尽可能全局唯一,
                              以方便您在无法正常收到消息情况下，可通过阿里云服务器管理控制台查询消息并补发
                              注意：不设置也不会影响消息正常收发
                      body:    Body 可以是任何二进制形式的数据， MQ 不做任何干预，
                               需要 Producer 与 Consumer 协商好一致的序列化和反序列化方式
                 */
            Message msg = new Message("TopicTest", "TagA", "keys", ("测试RocketMQ" + i).getBytes("UTF-8")
            );
            try {
            //发送同步消息
                SendResult sendResult = producer.send(msg);
                System.out.printf("%s%n", sendResult);

            } catch (Exception e) {
                e.printStackTrace();
                // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
                System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());
                Thread.sleep(1000);
            }
        }
        producer.shutdown();
    }
}

```

## 异步发送

```
package com.linjing.demo.message.asyn;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;

import java.io.UnsupportedEncodingException;
import java.util.Date;

/**
 * @author cxc
 * @date 2018/10/18 17:07
 * rockerMq发送异步消息
 */
public class AsynProducer {

    public static void main(String[] args) throws MQClientException, InterruptedException, UnsupportedEncodingException {
        DefaultMQProducer producer = new DefaultMQProducer("producer_demo");
        producer.setNamesrvAddr("112.74.43.136:9876");
        producer.start();

        for (int i = 0; i <= 100; i++) {
            Message msg = new Message("TopicTest", "TagA", "keys", ("测试RocketMQ" + i).getBytes("UTF-8")
            );
            try {
                // 异步发送消息, 发送结果通过 callback 返回给客户端。
                producer.send(msg, new SendCallback() {
                    @Override
                    public void onSuccess(SendResult sendResult) {
                        // 消费发送成功
                        System.out.println("SUCCESS信息:" + sendResult.toString());
                        System.out.println("send message success. topic=" + sendResult.getRegionId() + ", msgId=" + sendResult.getMsgId());
                    }

                    @Override
                    public void onException(Throwable throwable) {
                        // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
                        System.out.println("FAIL信息:" + throwable.getMessage());
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
                // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
                System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());
                Thread.sleep(1000);
            }
        }
        producer.shutdown();
    }
}

```

## 单向发送

```
package com.linjing.demo.message.oneway;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;

import java.io.UnsupportedEncodingException;
import java.util.Date;

/**
 * @author cxc
 * @date 2018/10/18 17:07
 * rockerMq发送单向消息
 */
public class OneWayProducer {

    public static void main(String[] args) throws MQClientException, InterruptedException, UnsupportedEncodingException {
        DefaultMQProducer producer = new DefaultMQProducer("producer_demo");
        producer.setNamesrvAddr("112.74.43.136:9876");
        producer.start();

        for (int i = 0; i <= 10; i++) {

            Message msg = new Message("TopicTest", "TagA", "keys", ("测试RocketMQ" + i).getBytes("UTF-8")
            );
            try {
                // 由于在 oneway 方式发送消息时没有请求应答处理，一旦出现消息发送失败，则会因为没有重试而导致数据丢失。若数据不可丢，建议选用可靠同步或可靠异步发送方式。
                producer.sendOneway(msg);
            } catch (Exception e) {
                e.printStackTrace();
                // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
                System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());
                Thread.sleep(1000);
            }
        }
        producer.shutdown();
    }
}

```

## 有序消息

```
package com.linjing.demo.message.sequence;

import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;

import java.io.UnsupportedEncodingException;
import java.util.Date;
import java.util.List;

/**
 * @author cxc
 * @date 2018/10/18 17:07
 * rockerMq发送顺序消息
 */
public class SequenceProducer {

    public static void main(String[] args) throws MQClientException, InterruptedException, UnsupportedEncodingException {
        DefaultMQProducer producer = new DefaultMQProducer("producer_demo");
        producer.setNamesrvAddr("112.74.43.136:9876");
        producer.start();

        for (int i = 0; i <= 100; i++) {
            int orderId = i % 10;
            Message msg = new Message("TopicTest", "TagA", "keys", ("测试RocketMQ" + i).getBytes("UTF-8")
            );
            try {
                // 顺序发送消息。
                producer.send(msg, new MessageQueueSelector() {
                            @Override
                            public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                                // arg的值其实就是orderId
                                Integer id = (Integer) o;
                                // mqs是队列集合，也就是topic所对应的所有队列
                                int index = id % list.size();
                                // 这里根据前面的id对队列集合大小求余来返回所对应的队列
                                return list.get(index);
                            }
                        }
                        , orderId);
            } catch (Exception e) {
                e.printStackTrace();
                // 消息发送失败，需要进行重试处理，可重新发送这条消息或持久化这条数据进行补偿处理
                System.out.println(new Date() + " Send mq message failed. Topic is:" + msg.getTopic());
                Thread.sleep(1000);
            }
        }
        producer.shutdown();
    }
}


```

# 延时消息

延时消息，简单来说就是当 producer 将消息发送到 broker 后，会延时一定时间后才投递给 consumer 进行消费。

RcoketMQ的延时等级为：1s，5s，10s，30s，1m，2m，3m，4m，5m，6m，7m，8m，9m，10m，20m，30m，1h，2h。level=0，表示不延时。level=1，表示 1 级延时，对应延时 1s。level=2 表示 2 级延时，对应5s，以此类推。

这种消息一般适用于消息生产和消费之间有时间窗口要求的场景。比如说我们网购时，下单之后是有一个支付时间，超过这个时间未支付，系统就应该自动关闭该笔订单。那么在订单创建的时候就会就需要发送一条延时消息（延时15分钟）后投递给 consumer，consumer 接收消息后再对订单的支付状态进行判断是否关闭订单。

设置延时非常简单，只需要在Message设置对应的延时级别即可：

```
Message msg = new Message("TopicTest",// topic
                        "TagA",// tag
                        ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)// body
                );
                // 这里设置需要延时的等级即可
                msg.setDelayTimeLevel(3);
                SendResult sendResult = producer.send(msg);
                
```