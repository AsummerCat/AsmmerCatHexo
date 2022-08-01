---
title: Yarn容量调度器多队列生成创建使用六
date: 2022-08-01 16:15:20
tags: [大数据,hadoop,yarn]
---
# Yarn容量调度器多队列生成创建使用

默认调度器只有一个队列(不满足生产环境使用),
多任务情况下,无法满足

# 多队列创建

### 创建规则
1.可根据调用的框架划分
每个框架的任务放入到指定的队列 (使用不多)

2.按照业务场景划分
登录注册,购物车,下单,业务部门1 ....

<!--more-->

### 创建多队列的好处
1.避免操作失误,导致所有资源全部耗尽

2.实现任务的降级使用,特殊使其保证重要的任务队列资源充足

# 参数中->配置多队列的容器调度器
案例: 新增一个`hive`队列

### 在`capacity-scheduler.xml` 中配置如下:
```
<!--指定多队列,增加hive队列,在原有的value中追加-->
<property>
   <name>yarn.scheduler.capacity.root.queues</name>
   <value>defalut,hive</value>
<property>

<!--降低default队列资源额定容量为40%,默认100%-->
<property>
   <name>yarn.scheduler.capacity.root.default.capacity</name>
   <value>40</value>
<property>

<!--降低default队列资源最大容量为60%,默认100%-->
<property>
   <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
   <value>60</value>
<property>

<!--指定hive队列资源额定容量为60%,-->
<property>
   <name>yarn.scheduler.capacity.root.hive.capacity</name>
   <value>40</value>
<property>


<!--用户最多可以使用hive队列多少资源 1表示 占用该队列的百分比 1为全部  0.0-1.0范围-->
<property>
   <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
   <value>1</value>
<property>

<!--是否开启当前队列投入使用-->
<property>
   <name>yarn.scheduler.capacity.root.hive.state</name>
   <value>RUNNING</value>
<property>

<!--是否指定某个用户可以使用该队列 * 表示任意用户-->
<property>
   <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
   <value>*</value>
<property>

<!--是否指定某个用户拥有该队列的操作权限(查看,删除,kill) * 表示任意用户-->
<property>
   <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
   <value>*</value>
<property>


<!--表示哪些用户,可以设置该队列的优先级->
<property>
   <name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</name>
   <value>*</value>
<property>



以下两个参数:
任务的超时时间设置:
yarn application -appId APPId -updateLifetime Timeout
如果不设置这个的话 
application的任务执行时间超过了默认的队列生命周期时间,立即kill;
如果设置的话
则按照当前队列的最大生命周期进行比较,超过则kill


<!--当前队列的最大生命周期, -1为永久-->
<property>
   <name>yarn.scheduler.capacity.root.hive.maximum-application-lifetime</name>
   <value>-1</value>
<property>

<!--当前队列的默认生命周期, -1为永久-->
<property>
   <name>yarn.scheduler.capacity.root.hive.default-application-lifetime</name>
   <value>-1</value>
<property>


```
这样设置之后,
default队列默认40, 最大借用后为60;
hive队列默认60, 最大借用后为80;

## 最后需要重启yarn或者执行刷新队列操作

```
yarn rmadmin -refreshQueues

```
刷新队列后,就可以看到这两条队列了


## 多队列的操作使用

### 1.使用命令行运行
运行jar的时候指定 使用哪个队列
` -D mapreduce.job.queuename=hive`
```
hadoop jar xxx.jar 启动类全类名 -D mapreduce.job.queuename=hive /input /output
```

### 2.使用代码job执行
```
//指定运行队列
conf.set("mapreduce.job.queuename","hive");
```


