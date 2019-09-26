---
title: dubbo开启qos功能命令行操作
date: 2019-09-26 14:48:42
tags: [SpringBoot,dubbo]
---

# dubbo开启qos功能

[文档地址](http://dubbo.apache.org/zh-cn/docs/user/references/qos.html)

# 作用

dubbo `2.5.8` 新版本增加了 QOS 模块，提供了新的 telnet 命令支持。

<!--more-->

QoS提供了一些启动参数，来对启动进行配置，他们主要包括：

| 参数               | 说明              | 默认值 |
| ------------------ | ----------------- | ------ |
| qosEnable          | 是否启动QoS       | true   |
| qosPort            | 启动QoS绑定的端口 | 22222  |
| qosAcceptForeignIp | 是否允许远程访问  | false  |

> 注意，从2.6.4/2.7.0开始，qosAcceptForeignIp默认配置改为false，如果qosAcceptForeignIp设置为true，有可能带来安全风险，请仔细评估后再打开。







# 两种方式开启:

### 使用XML方法进行配置

如果要通过XML配置响应的QoS相关的参数，可以进行如下配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
  <dubbo:application name="demo-provider">
    <dubbo:parameter key="qos.enable" value="true"/>
    <dubbo:parameter key="qos.accept.foreign.ip" value="false"/>
    <dubbo:parameter key="qos.port" value="33333"/>
  </dubbo:application>
  <dubbo:registry address="multicast://224.5.6.7:1234"/>
  <dubbo:protocol name="dubbo" port="20880"/>
  <dubbo:service interface="org.apache.dubbo.demo.provider.DemoService" ref="demoService"/>
  <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl"/>
</beans>
```

### 使用spring-boot自动装配方式配置

如果是spring-boot的应用，可以在`application.properties`或者`application.yml`上配置:

```
dubbo.application.qosEnable=true
dubbo.application.qosPort=33333
dubbo.application.qosAcceptForeignIp=false
```



# 使用

## 连接命令:

```java
telnet localhost 22222
```

## ls  列出消费者和提供者

```
dubbo>ls
As Provider side:
+----------------------------------+---+
|       Provider Service Name      |PUB|
+----------------------------------+---+
|org.apache.dubbo.demo.DemoService| N |
+----------------------------------+---+
As Consumer side:
+---------------------+---+
|Consumer Service Name|NUM|
+---------------------+---+

```

### Online 上线服务命令

```
//上线所有服务
dubbo>online
OK

//根据正则，上线部分服务
dubbo>online com.*
OK
```

## Offline 下线服务命令

```
//下线所有服务
dubbo>offline
OK

//根据正则，下线部分服务
dubbo>offline com.*
OK
```

## help 命令

```java
//列出所有命令
dubbo>help

//列出单个命令的具体使用情况
dubbo>help online
+--------------+----------------------------------------------------------------------------------+
| COMMAND NAME | online                                                                           |
+--------------+----------------------------------------------------------------------------------+
|      EXAMPLE | online dubbo                                                                     |
|              | online xx.xx.xxx.service                                                         |
+--------------+----------------------------------------------------------------------------------+


```