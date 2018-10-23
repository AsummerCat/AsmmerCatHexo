---
title: linux下RocketMQ的安装
date: 2018-10-18 15:43:56
tags: [RocketMQ,消息队列,linux]
---
[RocketMQ的基本概念](https://blog.csdn.net/qq_32711825/article/details/78579864)
[错误参考](https://blog.csdn.net/a906423355/article/details/78192828)

# 官网下载RocketMQ

点击这里 **[官网](http://rocketmq.apache.org/docs/quick-start/)**

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/4.3.0/rocketmq-all-4.3.0-source-release.zip
```
<!--more-->

# 进入目录解压
## 首先需要安装zip解压工具
`yum -y install zip unzip`

---

## 解压
```
  > unzip rocketmq-all-4.3.0-source-release.zip
  > cd rocketmq-all-4.3.0/
  > mvn -Prelease-all -DskipTests clean install -U
  > cd distribution/target/apache-rocketmq
  进入编译后目录
```
---

# 配置一下环境变量


```
vi /etc/profile
添加:
export rocketmq=/home/rocketmq-all-4.3.1/distribution
export PATH=$PATH:$rocketmq/bin

保存:
source /etc/profile
```

---

# 修改防火墙 安全组

```
打开 10909 9876  10911端口
```

---

# Start Name Server

>它是一个几乎无状态节点，可集群部署，节点之间无任何信息同步。（2.X版本之前rocketMQ使用zookeeper做topic路由管理）。Name Server 是专为 RocketMQ设计的轻量级名称服务，代码小于1000行，具有简单、可集群横吐扩展、无状态等特点。将要支持的主备自动切换功能会强依赖 Name Server。


```
  > nohup sh bin/mqnamesrv &
  > tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
```

---

# Start Broker

>Broker 部署相对复杂，Broker分为Master与Slave，一个Master可以对应多个Slave，但是一个Slave只能对应一个Master，Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slave。Master也可以部署多个。每个Broker与Name Server 集群中的所有节点建立长连接，定时注册Topic信息到所有Name Server。

```
  > nohup sh mqbroker -n localhost:9876 autoCreateTopicEnable=true &
  或者
  >nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 &  
  > tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 172.30.30.233:10911] boot success..
```

---

# 关机服务器

```
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK

```

---

# Note：向MQ发送和接收消息(验证失败)

## 通过java代码实现的案例生产者生产消息

　```
　$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer 
　```

## 通过java代码实现案例消费者消费消息

　`　$ sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
`
## 监控MQ，调用命令监控在target的bin目录下

　`　$ sh mqadmin clusterList -n localhost:9876`

---

## 单机部署

修改服务器中broker的配置，添加服务器IP(公网)
目录在 :`/conf/broker.conf`

```
新增加一下内容:

brokerIP1=xx.xx.xx.xx  # 你的公网IP
namesrvAddr=IP:端口   默认是9876
```
 然后重启  
 注意，重点是: `-c conf/broker.conf`
 
 ```
   > nohup sh bin/mqbroker -n localhost:9876 -c conf/broker.conf &
  > tail -f ~/logs/rocketmqlogs/broker.log 
  The broker[%s, 112.74.43.136:10911] boot success..
 ```

---

# 注意事项 

## 在虚拟机里面可能因为rockrtmq需要的java内存比较大 可能启动报错
>“VM warning: INFO: OS::commit_memory(0x00000006c0000000, 2147483648, 0) faild; error=’Cannot allocate memory’ (errno=12)”

解决方案:


>修改编译后的/bin中的服务启动脚本 runserver.sh 、runbroker.sh 中对于内存的限制，​改成如下示例：

```
JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=128m"

```

## UseCMSComactAtFullCollection is deprecated and will likely be removed in a future release.java.net.BindException: 地址正在使用？

问题原因：上述情况是系统存在多个Jdk同时使用导致的，如本文开头安装maven时讲到的冲突问题

解决方案：删除/停止多余jdk，保证系统使用唯一jdk源。

## 本地网络连接正常，（linux）联网失败，ping不通？ 

解决方案：修改 /etc/resolv.conf dns 文件 添加 nameserver 8.8.8.8 && 8.8.4.4

## 有个问题打死连接不上broker 最后把地址改到 targer下运行bin里的 就可以了

不知道是不是修改了这个 maven编译问题???

```
JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=128m"
```

## javaDemo中无法连接到服务器地址

第一种: org.apache.rocketmq.remoting.exception.RemotingTooMuchRequestException: sendDefaultImpl call timeout

解决:  

```
修改 /conf/中的broker.conf 添加 ip地址和namesrvAddr
```

第二种:  
No route info of this topic

解决:  

```
在启动mqbroker的时候需要指定autoCreateTopicEnable=true

例如：
nohup sh mqbroker -n 112.74.43.136:9876 autoCreateTopicEnable=true > ~/logs/rocketmqlogs/broker.log 2>&1 &

```

第三种:  
 可能是因为装了docker 启动后 被虚拟网卡占用 引入到 172的那个地址了
 
 解决: 
 
 ```
 关闭docker 
 systemctl stop docker
 
 ```
