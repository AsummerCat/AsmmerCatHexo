---
title: sentinel的使用及其对feign的支持
date: 2020-05-19 20:50:57
tags: [SpringCloudAlibaba,sentinel,feign]
---

# sentinel基础配置

## 基础知识点

```
1.限流
2.降级
3.热点key

在设置热点key的热点规则的时候

0代表第一个参数 可以设置当前反腐如果携带第一个参数的话进行控制
还可以配置高级规则 则指定第一个参数位等于XX的时候进行另外的控制
```
<!--more-->

## @SentinelResource注解的使用
需要注意的是: ==注解方式不支持private方法==

```
@SentinelResource(
Value="埋点key",
blockHandlerClass=自定义的限流处理类.class(也是静态方法),
blockHandler="自定义的限流方法",
fallback="用于在抛出异常的时候提供降级的逻辑处理,需要注意的是可以针对所有类型的异常 (除了exceptionsToIgnore里面排除掉的)",
fallbackClass=自定义的降级处理类.class(也是静态方法),
defaultFallback="默认的fallback",
exceptionsToIgnore={IllegalArgumentException.class,2.class}
)

如果defaultFallback和fallback同时存在 这fallback生效

例如: 降级类
需要注意的是这里是静态方法
public class CustomerBlockHandler
{
    public static String handleException(BlockException exception){
        return "限流中";
    }
    
}

```
==exceptionsToIgnore={IllegalArgumentException.class,2.class}==

==表示:如果出现该异常,则不再有fallback方法进行兜底,没有降级效果==


## 与feign整合

### yml配置开启
```
跟hystrix一样 需要开启支持

feign:
  sentinel: 
    enable: true
```
### 开启降级方法
统一处理异常
```
@feignClient(value="xxxx",fallback=xxxx降级.class)
```

## 持久化
与nacos整合
```
添加sentinel-datasource-nacos的pom

```
### 修改bootstrap.yml
```
# 连接sentinel监控台
spring.cloud.sentinel.transport.dashboard=localhost:8080

# 连接nacos注册中心 拉取数据
spring.cloud.sentinel.datasource.ds.nacos.server-addr=localhost:8848
spring.cloud.sentinel.datasource.ds.nacos.dataId=${spring.application.name}-sentinel
spring.cloud.sentinel.datasource.ds.nacos.groupId=DEFAULT_GROUP
spring.cloud.sentinel.datasource.ds.nacos.rule-type=flow
```
### 数据格式:
```
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
### 数据含义:
```
可以看到上面配置规则是一个数组类型，数组中的每个对象是针对每一个保护资源的配置对象，每个对象中的属性解释如下：

resource：资源名，即限流规则的作用对象
limitApp：流控针对的调用来源，若为 default 则不区分调用来源
grade：限流阈值类型（QPS 或并发线程数）；0代表根据并发数量来限流，1代表根据QPS来进行流量控制
count：限流阈值
strategy：调用关系限流策略
controlBehavior：流量控制效果（直接拒绝、Warm Up、匀速排队）
clusterMode：是否为集群模式
```