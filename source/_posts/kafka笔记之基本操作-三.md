---
title: kafka笔记之基本操作三
date: 2020-10-09 04:04:07
tags: [kafka笔记]
---

# 命令行操作
## 创建一个topic (主题)
```
sh kafka-topics.sh --create --zookeeper 192.168.1.1:2181 --replication-factor 1 --partitions 1 --topic test

replication-factor: 副本机制
partitions: 分区
topic: 主题名称
```

## 查看topic列表
```
sh kafka-topics.sh --list --zookeeper localhost:2181

```
<!--more-->

## 发送消息
```
sh kafka-console-prodecer.sh --broker-list localhost:2181 --topic test 

出现下一行后 输入消息 

--broker-list: 发送消息的节点
```

## 接收消息(消费者)
```
sh kafka-console-prodecer.sh --bootstrap-server localhost:9092 --topic test --from-beginning 


--from-beginning: 从头开始消费消息

```

# 基于java-api实现
## 导入jar包 kafka-client
```
<dependency>
 <groupId>org.apache.kafka</groupId>
 <artifactId>kafka-clients</artifactId>
 <version>1.0.0</version>
</dependency>
```

## 创建生产者 KafkaProducer
```
Properties properties=new Properties();

//这边是连接到kafka集群的地址
properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG."192.168.1.1:9092,192.168.1.2:9092,192.168.1.3:9092");
//以下是生产者常用配置信息

//设置当前客户端ID
properties.put(ProducerConfig.CLIENT_ID_CONFIG,"KafkaDemo");

//设置确认机制
//0.发送给broker以后,不需要确认 (性能较高,但是会出现数据丢失)
//1.只需要kafka集群的leader节点确认即可返回
//-1. (ALL)需要ISR中的所有副本进行确认(需要集群中的所有节点确认,性能最低),也可能出现数据丢失,比如isr中节点只有一个的时候 
properties.put(ProducerConfig.ACKS_CONFIG,"-1");

//设置key的序列化方式 kafka内置有几种序列化方式 这边先选择INT序列化
properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.IntegerSerializer");

//设置value的序列化方式 kafka内置有几种序列化方式 这边先选择INT序列化
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.StringSerializer");

//producer对于同一个分区来说,会按照batch.size的大小进行(统一收集批量发送)
//默认16KB 
//如果配置了batch.size 并不会马上发送当前消息,而是做缓存等到了指定batch.size大小后 批量发送
properties.put(ProducerConfig.BATCH_SIZE_SIZE_CONFIG,"20kb");

//最大的请求大小 默认1MB
properties.put(ProducerConfig.MAX_REQUEST_SIZE_CONFIG,"20kb");


//创建生产者 key为int   value为String
KafkaProducer<Integer,String> producer=new KafkaProducer<Integer,String>(properties);

```

## 发送消息
在kafka1.0以后,默认的client都是使用异步发送消息
```
String topic="主题名称"
String message="消息内容"

//同步发送消息 同步获取发送结果
producer.send(new ProducerRecord<Integer,String>(topic,message)).get();

//异步发送消息
producer.send(new ProducerRecord<Integer,String>(topic,message),new Callback{
 @Override
 public void onCompletion(RecordMetadata recordMetadata,Exception e){
     if(recordMetadata!=null){
         system.out.println("async-offset:"+recordMetadata.offset()+"->partition"+recordMetadata.partition());
     }
 }
});

```

## 创建消费者
```
Properties properties=new Properties();

//这边是连接到kafka集群的地址
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG."192.168.1.1:9092,192.168.1.2:9092,192.168.1.3:9092");
//以下是消费者常用配置信息

//设置当前客户端ID 消费组的概念 同组竞争,不同组会同步消费1条消息
properties.put(ConsumerConfig.GROUP_ID_CONFIG,"KafkaConsumerDemo");

//是否开启自动提交
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG,"true");

//自动提交的间隔 毫秒
properties.put(ConsumerConfig.ENABLE_AUTO_INTERVAL_CONFIG,"1000");


//设置key的反序列化方式 kafka内置有几种反序列化方式 
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.IntegerDeserializer");

//设置value的反序列化方式
properties.put(ConsumerConfig.VALUE_SERIALIZER_CLASS_CONFIG,"org.apache.kafka.common.serialization.StringDeserializer");

//从哪里开始消费 默认是消费端的启动之后接收的消息,
//以下设置从头开始接收
properties.put(ConsumerConfig.AUTO
_OFFSET_RESET_CONFIG,"earliest");

//每次拉取的最大消息数 可参照消费者的消费能力
properties.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG,"100");




//创建消费者 
KafkaConsumer kafkaConsumer=new KafkaConsumer(properties);
//去订阅消息
kafkaConsumer.subscribe(Collections.SingletonList("主题名称"));

```

## 接收消息
```
while(true){
    //消费者主动拉取数据  设置超时时间
   ConsumerRecords<Integer,String>  consumerRecord= kafkaConsumer.pull(1000);
   
   //遍历获取消息
   for ( ConsumerRecord record:consumerRecord){
       System.out.println("message is"+record.value());
   }
   
}
```

## <font color="red">消费端配置groupID  生产者配置 clientId</font>

