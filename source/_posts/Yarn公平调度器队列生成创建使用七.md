---
title: Yarn公平调度器队列生成创建使用七
date: 2022-08-01 16:15:45
tags: [大数据,hadoop,yarn]
---
# Yarn公平调度器队列生成创建使用

公平调度器的配置涉及到两个文件 ,
一个是`yarn-site.xml`,
另外一个是公平调度器队列分配文件`fair-scheduler.xml`(文件名可自定义)
<!--more-->


## 配置`yarn-site.xml`文件,加入以下参数
```
<property>
  <name>yarn.resourcemanager.scheduler.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
  <description>配置使用公平调度器</description>
</property>

<property>
  <name>yarn.scheduler.fair.allocation.file</name>
  <value>xxxx路径下的/fair-scheduler.xml</value>
  <description>指明公平调度器队列分配配置文件</description>
</property>

<property>
  <name>yarn.scheduler.fair.preemption</name>
  <value>false</value>
  <description>禁止队列间资源抢占</description>
</property>
```
## fair-scheduler.xml参数可参考官网配置
或者参考CDH上的参考配置