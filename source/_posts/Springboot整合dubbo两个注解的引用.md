---
title: SpringBoot整合dubbo两个注解的引用
date: 2019-09-26 14:48:29
tags: [SpringBoot,dubbo]
---

# dubbo两个注解的引用

## 参数详细解释

[官网文档](http://dubbo.apache.org/zh-cn/docs/user/references/xml/dubbo-service.html)

首先需要导入

```
        <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.2.0</version>
        </dependency>
```

<!--more-->

# @ EnableDubbo

开启dubbo

# @Service

用在服务提供者中，在类或者接口中声明。
服务提供者实现相关的服务接口，当消费端调用相关的类时，最终会调用提供者的实现方法。

```
import org.apache.dubbo.config.annotation.Service;
```

### 参数详细解释

```
version        ->服务版本，建议使用两位数字版本，如：1.0，通常在接口不兼容时版本号才需要升级

group          ->服务分组，当一个接口有多个实现，可以用分组区分 默认""

delay          ->延迟注册服务时间(毫秒) ，设为-1时，表示延迟到Spring容器初始化完成时暴露服务

timeout        ->远程服务调用超时时间(毫秒)

retries        ->远程服务调用重试次数，不包括第一次调用，不需要重试请设为0

connections    ->对每个提供者的最大连接数，rmi、http、hessian等短连接协议表示限制连接数，dubbo等长连接协表示                  建立的长连接个数

loadbalance    ->负载均衡策略，可选值：random,roundrobin,leastactive，分别表示：随机，轮询，最少活跃调用

async          ->是否缺省异步执行，不可靠异步，只是忽略返回值，不阻塞执行线程

mock           ->设为true，表示使用缺省Mock类名，即：接口名 + Mock后缀，服务接口调用失败Mock实现类，该Mock                  类必须有一个无参构造函数，与Local的区别在于，Local总是被执行，而Mock只在出现非业务异常(比如                  超时，网络异常等)时执行，Local在远程调用之前执行，Mock在远程调用后执行。
                 需要注意的是 这个需要写在接口同级下
   
token          ->该参数接收两种类型值：boolean类型-True则生成UUID随机令牌，若为String则自定义令牌  

registry       ->向指定注册中心注册，在多个注册中心时使用，值为<dubbo:registry>的id属性，多个注册中心ID用逗                  号分隔，如果不想将该服务注册到任何registry，可将值设为N/A

provider       ->指定provider，值为<dubbo:provider>的id属性  类名

deprecated     ->服务是否过时，如果设为true，消费方引用时将打印服务过时警告error日志

accesslog      ->设为true，将向logger中输出访问日志，也可填写访问日志文件路径，直接把访问日志输出到指定文件

owner          ->服务负责人，用于服务治理，请填写负责人公司邮箱前缀

weight         ->服务权重

executes       ->服务提供者每服务每方法最大可并行执行请求数

proxy          ->生成动态代理方式，可选：jdk/javassist

cluster        ->集群方式，可选：failover/failfast/failsafe/failback/forking

filter         ->服务提供方远程调用过程拦截器名称，多个名称用逗号分隔

```

### mock详细

```
服务提供者不需要写啥内容
消费者
@Reference(group="hello",mock = "true") 
需要添加 mock =true

然后在接口同级的包下 创建一个类
例如
接口:
package com.linjingc.apidemo.service;

public interface SayService {

    public String sayHello();
}

mock类:
package com.linjingc.apidemo.service;

public class SayServiceMock implements SayService {
    public SayServiceMock() {
    }

    @Override
    public String sayHello() {
        return "请求失败";
    }
}

```

# @Reference

@Reference 用在消费端，表明使用的是服务端的什么服务

```
import org.apache.dubbo.config.annotation.Reference;
```

### 参数详细解释

```
group     ->服务分组，当一个接口有多个实现，可以用分组区分，必需和服务提供方一致

timeout   ->服务方法调用超时时间(毫秒)

retries   ->远程服务调用重试次数，不包括第一次调用，不需要重试请设为0

check     ->启动时检查提供者是否存在，true报错，false忽略

url       ->点对点直连服务提供者地址，将绕过注册中心

cache     ->以调用参数为key，缓存返回结果，可选：lru, threadlocal, jcache等

client    ->客户端传输类型设置，如Dubbo协议的netty或mina


```

### timeout详细

```
如果@Service设置了timeout 

@Reference 也设置了timeout  以Reference 为准

因为颗粒度最小

但是最好

在Provider上尽量多配置Consumer端属性
原因如下：

作服务的提供者，比服务使用方更清楚服务性能参数，如调用的超时时间，合理的重试次数，等等
在Provider配置后，Consumer不配置则会使用Provider的配置值，即Provider配置可以作为Consumer的缺省值。否则，Consumer会使用Consumer端的全局设置，这对于Provider不可控的，并且往往是不合理的
PS: 配置的覆盖规则：1) 方法级配置别优于接口级别，即小Scope优先 2) Consumer端配置 优于 Provider配置 优于 全局配置，最后是Dubbo Hard Code的配置值（见配置文档）
```



# 两边参数大部分都是差不多了