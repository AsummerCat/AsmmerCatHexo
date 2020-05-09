---
title: openfeign实现负载均衡
date: 2020-05-09 14:48:46
tags: [SpringCloud,feign]
---

# openfeign默认等待1秒钟超时

使用:

### 1.开启@enableFeignClient注解

### 2.使用@FeignClient(value="服务名称" )

### 3.修改yml配置文件 修改超时时间

```java
 设置feign客户端超时时间(OpenFeign默认支持ribbon)      r
 ibbon:  
   MaxAutoRetries: 1
   #同一台实例最大重试次数,不包括首次调用  
   MaxAutoRetriesNextServer: 1 
   #重试负载均衡其他的实例最大重试次数,不包括首次调用   
   OkToRetryOnAllOperations: false 
   # 对所有的操作请求都进行重试，如果是get则可以，如果是post,put等操作没有实现幂等的情况下是很危险的，所以设置为false   
   ConnectTimeout: 1000 #请求连接的超时时间   
   ReadTimeout: 5000 #请求处理的超时时间
```

<!--more-->

### 4.开启Feign的日志

```java
NONE : 默认的 ,不显示任何日志
BASIC: 仅记录请求方法,url,响应状态码及执行时间
HEADERS: 除了BASIC中定义的信息之外,请求请求和响应的头消息
FULL: 除了HEADERS中定义的信息之外,还有请求和响应的正文及元数据
```

```java
开启方式:
    第一种:
@Bean
 Logger.Level.feignLoggerLevel()
 {
     returnLogger.Level.FULL;
 }   
   # 将Feign接口的日志级别设置成DEBUG，因为Feign的logger.Level只对DEBUG做出响应 
   yml配置:
   logging:
      level:
         yourproject.userClient: debug
         
  第二种
  直接使用yml配置
  ## “com.client.Client”为Feign接口的完整类名
## 将Feign接口的日志级别设置成DEBUG，因为Feign的logger.Level只对DEBUG做出响应
logging.level.com.client.Client = debug 
## “baidu”为feignName
feign.client.config.baidu.loggerLevel= full 
```

# 整合hystrix 做降级

## 需要yml 开启feign的hystrix

```java
feign:
    hystrix:
        enabled: true
```



## 在主启动类上加入开启hystrix的注解

```java
@EnableHystrix
```

## 使用

### 第一种->fallbackFactory

```java
@Service
@FeignClient(value = "TipProducerServer", fallbackFactory = TipsServerFallBackFactory.class)
public interface TipsService {

    @RequestMapping(value = "sendTip/{tip}")
    String sendTip(@PathVariable("tip") String tip);

    @RequestMapping(value = "index")
    String index();
}
```

降级工厂位置:

```java
@Component
public class TipsServerFallBackFactory implements FallbackFactory<TipsService> {
    private static final Logger logger = LoggerFactory.getLogger(TipsServerFallBackFactory.class);

    @Override
    public TipsService create(Throwable throwable) {
        logger.info("fallback reason was: {} ", throwable.getMessage());
        return new TipsService() {
            @Override
            public String sendTip(String tip) {
                return "sendTip请求失败";
            }

            @Override
            public String index() {
                return "index请求失败";
            }
        };

    }
}
```



### 第二种->fallback

```java
@Service
@FeignClient(value = "TipProducerServer", fallback = TipsServerFallBack.class)
public interface TipsService {

    @RequestMapping(value = "sendTip/{tip}")
    String sendTip(@PathVariable("tip") String tip);

    @RequestMapping(value = "index")
    String index();
}
```

降级方法位置:

```java

public class TipsServerFallBack implements TipsService {

    @Override
            public String sendTip(String tip) {
                return "sendTip请求失败";
            }

            @Override
            public String index() {
                return "index请求失败";
            }
}
```

