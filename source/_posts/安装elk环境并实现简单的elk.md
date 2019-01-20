---
title: 安装elk环境并实现简单的elk
date: 2019-01-20 17:28:09
tags: ELK日志分析框架
---

# 安装elk

使用版本    

```java
elasticsearch-2.4.6
kibana-4.6.1
logstash-2.4.0
```

<!--more-->

# 前言

elk   -> 当日500m内-1g日志

elk+filebeat ->  filebeat搜集日志后直接发送给logstash 适合 5g以上

felk+kafaka ->每日至少30g以上的日志

不建议加入redis 容易挂~

需要注意的是如果是用集群的话 搜集日志 最好带一个uuid 这样kabana里面可以更快速定位日志

[下载地址](https://www.elastic.co/downloads)

# 安装elasticsearch

**注意的是elasticsearch 自带document(json) 形式存储 支持TB级存储**

### 修改配置文件

`/config/elasticsearch.yml`

```java
# 集群名称
cluster.name: my-application 
# 节点名称
node.name: node-1 
# 该节点是否是master true 表示是  默认为true
node.master: true
# 该节点是否存储数据 默认为true
node.data: true
# Http访问端口 默认为9200 
http.port: 9200
# 节点之间通信端口 默认9300
transport.tcp.port: 9300
# 访问地址 配置外网访问
network.host: 0.0.0.0
# 负载均衡节点
discovery.zen.ping.unicast.hosts: ["host1", "host2"]
# 设置一台机子上运行的节点数目 默认 1 一般也只在一台机子上部署一台
node.max_local_storage_nodes: 1

```

其他配置文件 可以去看默认的elasticsearch.yml

### 安装elasticsearch可视化插件

```java
./bin/plugin install mobz/elasticsearch-head
```

访问路径:

```java
http://192.168.1.101:9200/_plugin/head/
```

注意要先启动elasticsearch 才能访问该地址

![](/img/2019-1-20/elasticsearch.png)



### 启动

运行

`./bin/elasticsearch`

` ./bin/elasticsearch -d 后台启动 `



# 安装logstash

注意 这个是每个需要搜集日志服务器都需要安装的

这边是没有默认的配置文件的 所以我们要手动创建一个配置文件

**Filebeat啊，根据input来监控数据，根据output来使用数据！！！**

 　对应于，**Logstash啊，有input、filter和output**

###  创建配置文件

自定义收集log规则

需要注意的是如果目录有 "-" 可能会有问题

```
input {
     stdin { } 
}
output {
    stdout { }
}
```

创建一个目录 config,创建一个简单的日志收集规则log.conf

```java
input {
        file {
        // 收集日志的目录 可以用* 
                path => "/var/log/elasticsearch/cluster.log"
        //       标记类型
                type => "elk-java-log"
        // 收集日志 的地方     beginning从头开始   end 在结尾开始收集    默认end 
                start_position => "beginning"
        //        隔多久检查一次被监听文件状态,默认1秒
                stat_interval => "2"
        // 规则  比如设置换行收集等~    
        codec插件，默认是plain，可以设置编码格式
        //      解决换行问题
                codec => multiline {
                //换行规则第一种:
                //      pattern => "^\["
                //换行规则第二种:
                        pattern => "^%{TIMESTAMP_IS08601}"
                        negate => true
                        what => "previous"
                }
        }
    beats {
        //收集日志的端口 如果是远程的话 需要注意 beats只能收集本机日志
        port =>5044
    }
}

output {
        if [type] == "elk-java-log" {
             // 连接 elasticsearch的地址
                elasticsearch {
                        hosts => ["192.168.1.101:9200"]
             // 收集日志       表示索引库 根据日期的小时来分片
                        index => "elk-java-log-%{+YYYY.MM.dd}"
             // 如果配置了nginx的密码则需要填写
                        user => user
                        password => password
                }
        }
}
```

推荐用这个！！！

因为，在调试，每次都要重启。加这个，不需每次去重启Logstash，即自己会加载。

添加--configtest参数检查配置语法是否有误！！！

```java
./bin/logstash -f ./config/log.conf --auto-reload --configtest
```

```
重启加载
./bin/logstash -f ./config/log.conf systemctl restart logstash
```

这样就收集到日志了

![](/img/2019-1-20/logstash.png)

# 安装kibana

### 修改配置文件

需要修改的地方并不多

```
# 服务地址
server.host: "192.168.80.10" 
# 端口号
server.port: 5601
# elasticsearch的地址
elasticsearch.url: "http://192.168.80.10:9200"
# Kibana使用索引Elasticsearch存储保存搜索,可视化
kibana.index: ".kibana"


对于server.host，最好别0.0.0.0，不安全。不建议
```

接着 进入 `./bin/kibana` 启动

### 初始化界面

访问地址  http://192.168.80.10:5601

进行相关配置

* 选择监听的日志

* 点击确认

  ![](/img/2019-1-20/kibana1.png)

![](/img/2019-1-20/elk-log)

这样我们就已经实现了一个简单的elk日志收集

点击搜索 关键字会高亮显示

# 注意实现

我们这边虽然实现了简单的elk 但是并不安全 所有人都可以访问

后继 我们将实现登录安全控制