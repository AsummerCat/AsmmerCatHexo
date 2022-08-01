---
title: SpringBoot使用Actuator进行健康监控
date: 2018-10-05 15:41:54
tags: [SpringBoot]
---

>参考:  
>1.[SpringBoot使用Actuator进行健康监控](https://blog.csdn.net/pengjunlee/article/details/80235390)


# 健康监控

# 添加POM依赖

```
<!-- spring-boot-监控-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```

<!--more-->

注意：若在核心配置文件中未添加 `management.security.enabled=false `配置，将会导致用户在访问部分监控地址时访问受限，报401未授权错误。

`application.yml`中指定监控的HTTP端口（如果不指定，则使用和Server相同的端口）；指定去掉某项的检查（比如不监控health.mail）：

```
server:
  port: 8083
management:
    port: 8083
    security:   
      enabled: false  #
```

---

# 监控和管理端点  

```
端点名           描述
autoconfig      所有自动配置信息（ positiveMatches :运行的， negativeMatches 未运行组件）
auditevents     审计事件
beans           所有Bean的信息
configprops     所有配置属性
dump            线程状态信息
env             当前环境信息
health          应用健康状况
info            当前应用信息
metrics         应用的各项指标
mappings        应用@RequestMapping映射路径
shutdown        关闭当前应用（默认关闭）
trace           追踪信息（最新的http请求）
heapdump        下载内存快照
```

Actuator监控项:  
![](/img/SpringBoot/2018-10-5/SpringBoot6.png)

http://localhost:8081/info 读取配置文件`application.properties`的 `info.*`属性

在InfoProperties 读取

application.properties :

```
info.app.version=v1.2.0
info.app.name=abc
#远程关闭开启
endpoints.shutdown.enabled=true  
#访问：http://localhost:8083/shutdown   关闭服务
```
---

# 自定义配置说明

```
#关闭metrics功能
endpoints.metrics.enabled=false
#开启shutdown远程关闭功能
endpoints.shutdown.enabled=true
#设置beansId
endpoints.beans.id=mybean
#设置beans路径
endpoints.beans.path=/bean
#关闭beans 功能
endpoints.beans.enabled=false
#关闭所有的
endpoints.enabled=false 
#开启单个beans功能
endpoints.beans.enabled=true
#所有访问添加根目录
management.context-path=/manage
​
management.port=8181


```
---
