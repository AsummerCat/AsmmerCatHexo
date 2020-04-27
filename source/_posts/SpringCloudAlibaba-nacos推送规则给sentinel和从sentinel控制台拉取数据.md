---
title: SpringCloudAlibaba-nacos推送规则给sentinel和从sentinel控制台拉取数据
date: 2020-04-27 10:54:17
tags: [SpringCloudAlibaba,sentinel,nacos]
---

# nacos推送数据给sentinel

## 需要打开nacos持久化到mysql(可有可无)

### 第一步

导入脚本

### 修改nacos的配置文件

```java
添加数据库的相关信息
```

<!--more-->

# 开始推送给sentinel

这里在代码中的埋点就很有效了

## 首先导入pom.xml sentinel-datasource-nacos

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
    <version>1.7.2</version>
</dependency>
```

## 修改bootstrap.yml

```java
# 连接sentinel监控台
spring.cloud.sentinel.transport.dashboard=localhost:8080

# 连接nacos注册中心 拉取数据
spring.cloud.sentinel.datasource.ds.nacos.server-addr=localhost:8848
spring.cloud.sentinel.datasource.ds.nacos.dataId=${spring.application.name}-sentinel
spring.cloud.sentinel.datasource.ds.nacos.groupId=DEFAULT_GROUP
spring.cloud.sentinel.datasource.ds.nacos.rule-type=flow
```

`Nacos`存储的具体配置类源码如下：

```java
public class NacosDataSourceProperties extends AbstractDataSourceProperties {

    private String serverAddr;
    private String groupId;
    private String dataId;

    // commercialized usage
    //下面4个参数是阿里云商业化产品使用的
    private String endpoint;
    private String namespace;
    private String accessKey;
    private String secretKey;

}
```

## 在nacos添加参数

json类型

`注意了count": 2,   在2020年4月27日17:19:431   1.71版本中可能会出现死循环 `

```java
[
    {
        "resource": "hello",
        "limitApp": "default",
        "grade": 1,
        "count": 2,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

```java
可以看到上面配置规则是一个数组类型，数组中的每个对象是针对每一个保护资源的配置对象，每个对象中的属性解释如下：

resource：资源名，即限流规则的作用对象
limitApp：流控针对的调用来源，若为 default 则不区分调用来源
grade：限流阈值类型（QPS 或并发线程数）；0代表根据并发数量来限流，1代表根据QPS来进行流量控制
count：限流阈值
strategy：调用关系限流策略
controlBehavior：流量控制效果（直接拒绝、Warm Up、匀速排队）
clusterMode：是否为集群模式
```

![nacos整合sentinel](/img/2020-04-26/nacos整合sentinel.png)

## 代码中埋点

```java
@SentinelResource(value = "hello",
			blockHandler = "block",
			fallback = "fallback")
```

可以动态切换限流规则



# sentinel修改数据推送给nacos

