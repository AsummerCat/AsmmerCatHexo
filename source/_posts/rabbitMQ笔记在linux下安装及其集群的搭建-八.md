---
title: rabbitMQ笔记在linux下安装及其集群的搭建(八)
date: 2020-09-12 14:39:36
tags: [rabbitMQ笔记]
---

# rabbitMQ笔记在linux下安装及其集群的搭建(八)
在安装(搭建集群)之前 确定两个点  
1:防火墙关掉    
2:打开网络  

<!--more-->

# 关闭防火墙

```
关闭防火墙 systemctl stop firewalld.service 禁止开机自启 systemctl disable firewalld.service
```

## 安装流程
进入/etc/yum.repos.d/ 文件夹

创建rabbitmq-erlang.repo 文件

内容如下
```
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=https://dl.bintray.com/rabbitmq-erlang/rpm/erlang/21/el/7
gpgcheck=1
gpgkey=https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```
创建rabbitmq.repo 文件

内容如下
```
[bintray-rabbitmq-server]
name=bintray-rabbitmq-rpm
baseurl=https://dl.bintray.com/rabbitmq/rpm/rabbitmq-server/v3.8.x/el/8/
gpgcheck=0
repo_gpgcheck=0
enabled=1
```
安装命令

`yum install rabbitmq-server`

## rabbitmq相关命令

开启  
`service rabbitmq-server start`  
关闭  
`service rabbitmq-server stop`  
查看状态  
`service rabbitmq-server status`  
重启  
`service rabbitmq-server restart`  
启用插件页面管理  
`rabbitmq-plugins enable rabbitmq_management`  
创建用户  
`rabbitmqctl add_user admin mypassword`  
赋予权限  
```
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```
浏览器访问http://机器IP:15672打开管理界面，使用上一步配置好的admin账号登录

## 或者采用以下方式下载安装
因为rabbitMQ是基于erlang开发的
```
下载erlang wget http://www.rabbitmq.com/releases/erlang/erlang-18.2-1.el6.x86_64.rpm 
安装erlang 
rpm -ihv http://www.rabbitmq.com/releases/erlang/erlang-18.2-1.el6.x86_64.rpm

或者:
'yum install erlang' 直接下载erlang

安装完erlang之后 开始rabbitmq 还是再提醒一下 先装erlang 再装rabbitmq
装Rabbitmq之前 先装一个公钥 :
rpm --import https://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
装好公钥之后 下载Rabbitmq:
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.6/rabbitmq-server-3.6.6-1.el7.noarch.rpm
安装:
rpm -ihv rabbitmq-server-3.6.6-1.el7.noarch.rpm
安装中途可能会提示你需要一个叫socat的插件
如果提示了 就先安装socat 再装rabbitmq
安装socat:
yum install socat
至此 就装好了Rabbitmq了 可以执行以下命令启动Rabbitmq:
service rabbitmq-server start
和windows环境下一样 rabbitmq对于linux也提供了他的管理插件
安装rabbitmq管理插件: rabbitmq-plugins enable rabbitmq_management
安装完管理插件之后 如果有装了浏览器的话 比如火狐 可以和windows一样 访问 一下 localhost:15672 可以看到一
个熟悉的页面
```

# rabbitmq集群搭建，配置

## 配置节点
```
rabbbitmq由于是由erlang语言开发的 天生就支持分布式
rabbitmq 的集群分两种模式 一种是默认模式 一种是镜像模式
当然 所谓的镜像模式是基于默认模式加上一定的配置来的
在rabbitmq集群当中 所有的节点（一个rabbitmq服务器） 会被归为两类 一类是磁盘节点 一类是内存节点
磁盘节点会把集群的所有信息(比如交换机,队列等信息)持久化到磁盘当中，而内存节点只会将这些信息保存到内存
当中 讲白了 重启一遍就没了。
为了可用性考虑 rabbitmq官方强调集群环境至少需要有一个磁盘节点， 而且为了高可用的话， 必须至少要有2个
磁盘节点， 因为如果只有一个磁盘节点 而刚好这唯一的磁盘节点宕机了的话， 集群虽然还是可以运作， 但是不能
对集群进行任何的修改操作（比如 队列添加，交换机添加，增加/移除 新的节点等）
具体想让rabbitmq实现集群， 我们首先需要改一下系统的hostname (因为rabbitmq集群节点名称是读取
hostname的)
这里 我们模拟3个节点 :
rabbitmq1
rabbitmq2
rabbitmq3
linux修改hostname命令: hostnamectl set-hostname [name]
```
## 加入节点
```
修改后重启一下 让rabbitmq重新读取节点名字
然后 我们需要让每个节点通过hostname能ping通（记得关闭防火墙） 这里 我们可以修改修改一下hosts文件
关闭防火墙:
关闭防火墙 systemctl stop firewalld.service
禁止开机自启 systemctl disable firewalld.service
接下来,我们需要将各个节点的.erlang.cookie文件内容保持一致(文件路径/var/lib/rabbitmq/.erlang.cookie)
因为我是采用虚拟机的方式来模拟集群环境， 所以如果像我一样是克隆的虚拟机的话 同步.erlang.cookie文件这个
操作在克隆的时候就已经完成了。
上面这些步骤完成之后 我们就可以开始来构建集群 了
我们先让rabbitmq2 加入 rabbitmq1与他构建为一个集群
执行命令( ram:使rabbitmq2成为一个内存节点 默认为:disk 磁盘节点)： 

'rabbitmqctl stop_app rabbitmqctl join_cluster rabbit@rabbitmq1 --ram rabbitmqctl start_app'

在构建的时候 我们需要先停掉rabbitmqctl服务才能构建 等构建完毕之后再启动
我们吧rabbitmq2添加完之后在rabbitmq3节点上也执行同样的代码 使他也加入进去 当然 我们也可以让
rabbitmq3也作为一个磁盘节点
```
## 有关集群的其他命令
```
rabbitmq-server -detached 启动RabbitMQ节点
rabbitmqctl start_app 启动RabbitMQ应用，而不是节点
rabbitmqctl stop_app 停止 
rabbitmqctl status 查看状态 
rabbitmqctl add_user mq 123456 rabbitmqctl
set_user_tags mq administrator 新增账户
rabbitmq-plugins enable rabbitmq_management 启用RabbitMQ_Management 
rabbitmqctl cluster_status 集群状态 
rabbitmqctl forget_cluster_node
rabbit@[nodeName] 节点摘除 
rabbitmqctl reset application 重置

```