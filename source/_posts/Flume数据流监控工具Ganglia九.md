---
title: Flume数据流监控工具Ganglia九
date: 2022-08-12 09:46:17
tags: [Flume,大数据]
---
# Flume数据流监控工具Ganglia
Ganglia 由 `gmond`、`gmetad `和` gweb` 三部分组成。

gmond（Ganglia Monitoring Daemon）是一种轻量级服务，安装在每台需要收集指标数
据的节点主机上。使用 gmond，你可以很容易收集很多系统指标数据，如 CPU、内存、磁盘、
网络和活跃进程的数据等。

gmetad（Ganglia Meta Daemon）整合所有信息，并将其以 RRD 格式存储至磁盘的服务。
<!--more-->
gweb（Ganglia Web）Ganglia 可视化工具，gweb 是一种利用浏览器显示 gmetad 所存储
数据的 PHP 前端。在 Web 界面中以图表方式展现集群的运行状态下收集的多种不同指标数
据

## 安装
```
sudo yum -y install ganglia-gmetad 
sudo yum -y install ganglia-web
sudo yum -y install ganglia-gmond
```

## 修改配置文件
#### /etc/selinux/config
```
sudo vim /etc/selinux/config
修改为 ->这样PHP才能运行
SELINUX=disabled
```
selinux 生效需要重启

#### ganglia.conf
`sudo vim
/etc/httpd/conf.d/ganglia.conf`
设置允许访问的主机地址
```
# 通过 windows 访问 ganglia,需要配置 Linux 对应的主机(windows)ip 地址
 Require ip 192.168.9.1


```
#### gmetad.conf
`sudo vim /etc/ganglia/gmetad.conf`
设置默认的数据源地址
```
data_source "my cluster" 192.168.9.1
```
#### gmond.conf
对应监控主机上
```
修改为：
cluster {
 name = "my cluster"
 owner = "unspecified"
 latlong = "unspecified"
 url = "unspecified"
}
udp_send_channel {
 # those IPs will be used to create the RRDs.
 # mcast_join = 239.2.11.71
 # 数据发送给 hadoop102
 host = hadoop102
 port = 8649
 ttl = 1
}
udp_recv_channel {
 # mcast_join = 239.2.11.71
 port = 8649
# 接收来自任意连接的数据
 bind = 0.0.0.0
 retry_bind = true
 # buffer = 10485760
}
```
## 启动
```
sudo systemctl start gmond
sudo systemctl start httpd
sudo systemctl start gmetad
```

## web界面
```
http://hadoop102/ganglia
```
ps:以上操作如果权限不足
```
sudo chmod -R 777 /var/lib/ganglia
```