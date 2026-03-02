---
title: Rocketmq集群监控
date: 2018-10-18 17:53:51
tags: [RocketMQ,消息队列]
---

# 安装可视化界面

```
it clone https://github.com/apache/rocketmq-externals.git
```

然后进入rocketmq-console的目录：

```
cd rocketmq-externals/rocketmq-console
```


执行以下命令对rocketmq-cosole进行打包，把他做成一个jar包：

```
mvn package -DskipTests
```

然后进入target目录下，可以看到一个jar包，接着执行下面的命令启动工作台：

```
java -jar rocketmq-console-ng-1.0.1.jar --server.port=8080 --rocketmq.config.namesrvAddr=127.0.0.1:9876
```

这里务必要在启动的时候设置好NameServer的地址，如果有多个地址可以用分号隔开，接着就会看到工作台启动了，然后就通过浏览
器访问那台机器的8080端口就可以了，就可以看到精美的工作台界面。

这样可以看到 集群中的内容  可以配置其他的内容非常方便