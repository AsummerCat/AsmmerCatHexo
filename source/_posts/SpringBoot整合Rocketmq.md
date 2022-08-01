---
title: SpringBoot整合Rocketmq
date: 2018-10-18 16:53:51
tags: [RocketMQ,消息队列,SpringBoot]
---

# 整合rocketmq有两种方式

第一种 导入rocketmq-client手动实现

第二种 导入rocketmq-spring-boot-starter 快速开发包实现

## [demo地址](https://github.com/AsummerCat/rocketmq-demo)

<!--more--> 

# 客户端接收消息的几种消息监听器

### 顶级接口

```
MessageListener
    ->子类继承接口 
       MessageListenerConcurrently 以下内容都是实现这个的
```

### MessageListenerConcurrently 用来监听并发      如:广播消息

```
public interface MessageListenerConcurrently extends MessageListener
messagelistener对象用于同时接收异步传递的消息
```

### MessageListenerOrderly  用来按顺序接收消息

```
messagelistener对象用于按顺序接收异步传递的消息。一个队列,一个
*线程
```

### starter包中有两个实现类

```
DefaultMessageListenerConcurrently

DefaultMessageListenerOrderly

```

# 第一种

## 导入相关pom.xml

```
<!--客户端-->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.3.1</version>
</dependency>

```

## 编写Producer

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

## 编写 Consumer

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

## 几种发送方式

### 拼接组装消息体

```
	/**
	 * 构造普通消息结构体
	 *
	 * @return
	 * @throws UnsupportedEncodingException
	 */
	private Message initMessage() throws UnsupportedEncodingException {
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
		Message msg = new Message("TopicTest", "TagA", "keys", ("测试RocketMQ").getBytes("UTF-8"));
		return msg;
	}
```

### 可靠同步发送

```
	/**
	 * 发送普通同步消息
	 */
	public void sendBasicMessage() throws Exception {
		DefaultMQProducer producer = initDefaultMQProducer();
		Message msg = initMessage();
		//发送同步消息
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
```

### 可靠异步发送

```
public void sendAsyncMessage() throws Exception {
		DefaultMQProducer producer = initDefaultMQProducer();
		Message msg = initMessage();
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
```

### 单向发送 (日志收集)

```
    /**
	 * 单向发送
	 */
	public void sendOnewayMessage() throws Exception {
		DefaultMQProducer producer = initDefaultMQProducer();
		Message msg = initMessage();
		// 由于在 oneway 方式发送消息时没有请求应答处理，一旦出现消息发送失败，则会因为没有重试而导致数据丢失。若数据不可丢，建议选用可靠同步或可靠异步发送方式。
		producer.sendOneway(msg);
	}
```

### 延时消息发送

```
	/**
	 * 发送延时消息
	 */
	public void sendDelayMessage() throws Exception {
		DefaultMQProducer producer = initDefaultMQProducer();
		Message msg = initMessage();

		// 这里设置需要延时的等级即可 3秒
		msg.setDelayTimeLevel(3);
		SendResult sendResult = producer.send(msg);
	}
```

### 事务消息发送

#### 创建一个事务监听器

```

/**
 * 创建事务监听实现类
 */
public class TransactionListenerImpl implements TransactionListener {
    private AtomicInteger transactionIndex = new AtomicInteger(0);

    private ConcurrentHashMap<String, Integer> localTrans = new ConcurrentHashMap<>();

    @Override
    public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
        int value = transactionIndex.getAndIncrement();
        int status = value % 3;
        localTrans.put(msg.getTransactionId(), status);
        return LocalTransactionState.UNKNOW;
    }

    @Override
    public LocalTransactionState checkLocalTransaction(MessageExt msg) {
        Integer status = localTrans.get(msg.getTransactionId());
        if (null != status) {
            switch (status) {
                case 0:
                    return LocalTransactionState.UNKNOW;
                case 1:
                    return LocalTransactionState.COMMIT_MESSAGE;
                case 2:
                    return LocalTransactionState.ROLLBACK_MESSAGE;
            }
        }
        return LocalTransactionState.COMMIT_MESSAGE;
    }
}
```

#### 发送事务信息

```
/**
	 * 发送事务消息
	 */
	public void sendTransactionMessage() throws Exception {
		//创建一个事务监听器
		TransactionListener transactionListener = new TransactionListenerImpl();
		//这里使用 事务的生产者 来发送消息
		TransactionMQProducer producer = new TransactionMQProducer("please_rename_unique_group_name");
		ExecutorService executorService = new ThreadPoolExecutor(2, 5, 100, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2000), new ThreadFactory() {
			@Override
			public Thread newThread(Runnable r) {
				Thread thread = new Thread(r);
				thread.setName("client-transaction-msg-check-thread");
				return thread;
			}
		});

		producer.setExecutorService(executorService);
		producer.setTransactionListener(transactionListener);
		producer.start();

		//发送消息
		try {
			Message msg = initMessage();
			//这里使用 事务生产者发送事务信息 如果抛出异常即回滚数据
			TransactionSendResult sendResult = producer.sendMessageInTransaction(msg, null);
			System.out.printf("%s%n", sendResult);

			Thread.sleep(10);
		} catch (MQClientException | UnsupportedEncodingException e) {
			e.printStackTrace();
		}
	}
```

### 顺序发送

RocketMQ提供使用先进先出算法的顺序消息实现。

将rockmq的消息发送 到路由到 top 下的同一个order  比如 A top 下的A queue 保证顺序消费 

客户端可以是个集群 如果A消费失败(需要幂等性处理),失败转给另外一个客户端做处理 

```
 /**
	 * 发送有序消息 MessageQueueSelector 自定义规则发送数据
	 * 默认策略 有三种
	 * SelectMessageQueueByHash   根据hash
	 * SelectMessageQueueByMachineRoom 根据机房随机
	 * SelectMessageQueueByRandom   随机选择消息队列 random方法
	 */
	public void sendOrderMessage() throws Exception {
		DefaultMQProducer producer = initDefaultMQProducer();
		Message msg = initMessage();
		// 顺序发送消息。

		//根据订单号
		int orderId = 100 % 10;
		producer.send(msg, new MessageQueueSelector() {
					@Override
					//重写选择器 根据自己的路由到某个队列中 保证顺序消费
					public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
						// arg的值其实就是orderId
						Integer id = (Integer) o;
						// list是队列集合，也就是topic所对应的所有队列
						int index = id % list.size();
						// 这里根据前面的id对队列集合大小求余来返回所对应的队列
						return list.get(index);
					}
				}
				, orderId);
	}
```

## 消费

### 普通集群消费

```
	/**
	 * 接收到普通消息
	 */
	public void receiveBasicMessage() throws Exception {
		DefaultMQPushConsumer consumer = initDefaultMQPushConsumer();
		//注册监听消息
		consumer.registerMessageListener(new MessageListenerOrderly() {
			@Override
			public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
				//关闭自动提交
				consumeOrderlyContext.setAutoCommit(false);
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
	}
```

### 广播消费

```
   /**
	 * 广播消费信息
	 */
	public void receiveBroadcastMessage() throws Exception {
		DefaultMQPushConsumer consumer = initDefaultMQPushConsumer();
		//设置为广播模式 默认为CLUSTERING模式
		//MessageModel.BROADCASTING用来接收广播消息
		consumer.setMessageModel(MessageModel.BROADCASTING);
		consumer.registerMessageListener(new MessageListenerOrderly() {
			@Override
			public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
				context.setAutoCommit(false);
				System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
				return ConsumeOrderlyStatus.SUCCESS;
			}
		});
	}
```





## 第二种starter包使用

### 导入starter包

```
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

### 修改application.yml 加入rockermq的相关配置

```
## application.properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=my-group
```

### 使用RocketMQTemplate模板

```
@Resource
    private RocketMQTemplate rocketMQTemplate;
```

## 发送消息篇

你可以在配置文件里定义 消息组

##  普通消息发送

不保证消息不重复 需要自己处理幂等性问题 默认重试2次

```
@Resource
    private RocketMQTemplate rocketMQTemplate;  	


        //返回结果
		SendResult sendResult;
		//定义一个topic
		String topic = "user-topic";

		//发送同步消息 需要注意的可能会存在重试的情况 需要解决重复的问题 默认2次
		sendResult = rocketMQTemplate.syncSend(topic, "Hello, World!");

		//发送普通消息 需要注意的可能会存在重试的情况 需要解决重复的问题 默认2次
		sendResult = rocketMQTemplate.syncSend(topic, MessageBuilder.withPayload("Hello, World! I'm from spring message").build());

		//发送普通序列化消息 需要注意的可能会存在重试的情况 需要解决重复的问题 默认2次
		sendResult = rocketMQTemplate.syncSend(topic, new User().setUserAge((byte) 18));

		//发送普通消息 带超时时间 默认2次
		sendResult = rocketMQTemplate.syncSend(topic, "Hello, World!", 10);

		//发送序列化消息 转换成json发送 需要注意的可能会存在重试的情况 需要解决重复的问题 默认2次
		sendResult = rocketMQTemplate.syncSend(topic, MessageBuilder.withPayload(
				new User().setUserAge((byte) 21).setUserName("Lester")).setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON_VALUE).build());

		//发送单向消息
		rocketMQTemplate.sendOneWay("topic-name", "send one-way message");

		//发送异步消息 并且回调
		rocketMQTemplate.asyncSend(topic, new User().setUserAge((byte) 18), new SendCallback() {
			@Override
			public void onSuccess(SendResult var1) {
				System.out.printf("发送成功=%s %n", var1);
			}

			@Override
			public void onException(Throwable var1) {
				System.out.printf("发送失败=%s %n", var1);
			}
		});


		//发送带tag的消息 组装tag的格式:  topic:tag
		rocketMQTemplate.convertAndSend(topic + ":tag0", "I'm from tag0");


		//批量发送消息 设置不同的KEYS
		List<Message> msgs = new ArrayList<Message>();
		for (int i = 0; i < 10; i++) {
			msgs.add(MessageBuilder.withPayload("Hello RocketMQ Batch Msg#" + i).
					setHeader(RocketMQHeaders.KEYS, "KEY_" + i).build());
		}

		SendResult sr = rocketMQTemplate.syncSend(topic, msgs, 60000);
		System.out.printf("--- Batch messages send result :" + sr);


		//发送消息,并且接收客户端的回复String类型
		String replyString = rocketMQTemplate.sendAndReceive(topic, "request string", String.class);


		//发送消息,并且接收客户端的回复User类型  并且根据 key进行顺序发送
		User requestUser = new User().setUserAge((byte) 9).setUserName("requestUserName");
		User replyUser = rocketMQTemplate.sendAndReceive(topic, requestUser, User.class, "order-id");

```

## 事务管理器 发送

### 自定义的事务RocketMQTemplate

```
/**
 * 定义了一个事务发送者的模板配置
 */
@ExtRocketMQTemplateConfiguration(nameServer = "${demo.rocketmq.extNameServer}")
public class ExtRocketMQTemplate extends RocketMQTemplate {
}
```

### 使用自定义的事务RocketMQTemplate

```
@Resource(name = "extRocketMQTemplate")
private RocketMQTemplate extRocketMQTemplate;
```

### 创建一个事务监听器

```
/**
 * 创建一个事务监听器
 *
 */
@Configuration
public class TransactionListener {
	@RocketMQTransactionListener
	class TransactionListenerImpl implements RocketMQLocalTransactionListener {
		private AtomicInteger transactionIndex = new AtomicInteger(0);

		private ConcurrentHashMap<String, Integer> localTrans = new ConcurrentHashMap<String, Integer>();

		@Override
		public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
			String transId = (String) msg.getHeaders().get(RocketMQHeaders.TRANSACTION_ID);
			System.out.printf("#### executeLocalTransaction is executed, msgTransactionId=%s %n",
					transId);
			int value = transactionIndex.getAndIncrement();
			int status = value % 3;
			localTrans.put(transId, status);
			if (status == 0) {
				// Return local transaction with success(commit), in this case,
				// this message will not be checked in checkLocalTransaction()
				System.out.printf("    # COMMIT # Simulating msg %s related local transaction exec succeeded! ### %n", msg.getPayload());
				return RocketMQLocalTransactionState.COMMIT;
			}

			if (status == 1) {
				// Return local transaction with failure(rollback) , in this case,
				// this message will not be checked in checkLocalTransaction()
				System.out.printf("    # ROLLBACK # Simulating %s related local transaction exec failed! %n", msg.getPayload());
				return RocketMQLocalTransactionState.ROLLBACK;
			}

			System.out.printf("    # UNKNOW # Simulating %s related local transaction exec UNKNOWN! \n");
			return RocketMQLocalTransactionState.UNKNOWN;
		}

		@Override
		public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
			String transId = (String) msg.getHeaders().get(RocketMQHeaders.TRANSACTION_ID);
			RocketMQLocalTransactionState retState = RocketMQLocalTransactionState.COMMIT;
			Integer status = localTrans.get(transId);
			if (null != status) {
				switch (status) {
					case 0:
						retState = RocketMQLocalTransactionState.UNKNOWN;
						break;
					case 1:
						retState = RocketMQLocalTransactionState.COMMIT;
						break;
					case 2:
						retState = RocketMQLocalTransactionState.ROLLBACK;
						break;
				}
			}
			System.out.printf("------ !!! checkLocalTransaction is executed once," +
							" msgTransactionId=%s, TransactionState=%s status=%s %n",
					transId, retState, status);
			return retState;
		}
	}

	@RocketMQTransactionListener(rocketMQTemplateBeanName = "extRocketMQTemplate")
	class ExtTransactionListenerImpl implements RocketMQLocalTransactionListener {
		@Override
		public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
			System.out.printf("ExtTransactionListenerImpl executeLocalTransaction and return UNKNOWN. \n");
			return RocketMQLocalTransactionState.UNKNOWN;
		}

		@Override
		public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
			System.out.printf("ExtTransactionListenerImpl checkLocalTransaction and return COMMIT. \n");
			return RocketMQLocalTransactionState.COMMIT;
		}
	}
}

```

## 使用事务模板发送消息

```
       /**
		 * 事务管理器 事务发送者 需要配置rabbitMq事务管理器
		 */
		//使用事务发送者发送普通消息 需要注意的可能会存在重试的情况 需要解决重复的问题
		sendResult = extRocketMQTemplate.syncSend(topic, MessageBuilder.withPayload("Hello, World!2222".getBytes()).build());

		//使用事务发送者发送事务消息
		Message msg = MessageBuilder.withPayload("extRocketMQTemplate transactional message " + 5).
				setHeader(RocketMQHeaders.TRANSACTION_ID, "KEY_" + 5).build();
		sendResult = extRocketMQTemplate.sendMessageInTransaction(
				topic, msg, null);
		System.out.printf("------ExtRocketMQTemplate send Transactional msg body = %s , sendResult=%s %n",
				msg.getPayload(), sendResult.getSendStatus());
```



## 消费者接收消息 和回复消息

### 使用 注解

```
    @Service
	@RocketMQMessageListener     
```

### 介绍

```
/**
	 * **********************************************************************
	 * ********************客户端接收的方式**********************************
	 * **********************************************************************
	 * 实现类:
	 * RocketMQReplyListener :  接收并回复
	 * RocketMQListener: 接收没回复
	 * RocketMQPushConsumerLifecycleListener: 可以设置消费的起点位置
	 *
	 * @RocketMQMessageListener 注解中可以加入很多参数 详情点击
	 *  设置接收顺序: :consumeMode 并发还是顺序
	 *  设置接收模型: messageModel 集群还是广播
	 */
```

### 使用

```
/**
	 * **********************************************************************
	 * ********************客户端接收的方式**********************************
	 * **********************************************************************
	 * 实现类:
	 * RocketMQReplyListener :  接收并回复
	 * RocketMQListener: 接收没回复
	 * RocketMQPushConsumerLifecycleListener: 可以设置消费的起点位置
	 *
	 * @RocketMQMessageListener 注解中可以加入很多参数 详情点击
	 *  设置接收顺序: :consumeMode 并发还是顺序
	 *  设置接收模型: messageModel 集群还是广播
	 */

	/**
	 * 接收 top为XX 分组为XX 路由规则为tagA的消息 用字节接收并回复
	 * RocketMQReplyListener
	 * 并回复
	 */
	@Service
	@RocketMQMessageListener(topic = "${demo.rocketmq.bytesRequestTopic}", consumerGroup = "${demo.rocketmq.bytesRequestConsumer}", selectorExpression = "${demo.rocketmq.tag}")
	public class ConsumerWithReplyBytes implements RocketMQReplyListener<MessageExt, byte[]> {

		@Override
		public byte[] onMessage(MessageExt message) {
			System.out.printf("------- ConsumerWithReplyBytes received: %s \n", message);
			return "reply message content".getBytes();
		}

	}

	/**
	 * 接收String消息
	 * RocketMQListener
	 */
	@Service
	@RocketMQMessageListener(topic = "${demo.rocketmq.topic}", consumerGroup = "string_consumer", selectorExpression = "${demo.rocketmq.tag}")
	public class StringConsumer implements RocketMQListener<String> {
		@Override
		public void onMessage(String message) {
			System.out.printf("------- StringConsumer received: %s \n", message);
		}
	}


	/**
	 * 接收消息
	 * 设置消费者 消费数据的开始时间
	 * RocketMQPushConsumerLifecycleListener
	 */
	@Service
	@RocketMQMessageListener(topic = "${demo.rocketmq.msgExtTopic}", selectorExpression = "tag0||tag1", consumerGroup = "${spring.application.name}-message-ext-consumer")
	public class MessageExtConsumer implements RocketMQListener<MessageExt>, RocketMQPushConsumerLifecycleListener {
		@Override
		public void onMessage(MessageExt message) {
			System.out.printf("------- MessageExtConsumer received message, msgId: %s, body:%s \n", message.getMsgId(), new String(message.getBody()));
		}

		@Override
		public void prepareStart(DefaultMQPushConsumer consumer) {

			//设置消费者消费起点
			//    CONSUME_FROM_LAST_OFFSET, 从上一个偏移量消费
			//    CONSUME_FROM_FIRST_OFFSET, 从第一个补偿中消费
			//    CONSUME_FROM_TIMESTAMP, 从时间戳开始消费
			consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_TIMESTAMP);
			//表示表示从现在开始消费
			consumer.setConsumeTimestamp(UtilAll.timeMillisToHumanString3(System.currentTimeMillis()));
		}
	}

	/**
	 * 接收消息并且回复给客户端消息
	 * RocketMQReplyListener
	 * @author 一只写BUG的猫
	 */
	@Service
	@RocketMQMessageListener(topic = "${demo.rocketmq.objectRequestTopic}", consumerGroup = "${demo.rocketmq.objectRequestConsumer}", selectorExpression = "${demo.rocketmq.tag}")
	public class ObjectConsumerWithReplyUser implements RocketMQReplyListener<User, User> {

		@Override
		public User onMessage(User user) {
			System.out.printf("------- ObjectConsumerWithReplyUser received: %s \n", user);
			User replyUser = new User();
			replyUser.setUserAge((byte) 10);
			replyUser.setUserName("replyUserName");
			return replyUser;
		}
	}
```

