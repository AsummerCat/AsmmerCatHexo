---
title: SpringCloud中eureka_instance为前缀的的常用配置参数说明
date: 2019-06-27 21:41:16
tags: [SpringCloud,eureka]
---

# Spring Cloud中eureka.instance为前缀的的常用配置参数说明



## 参数详情

eureka.instance

```
* preferIpAddress                  是否优先使用IP地址作为主机名的标识                   false

* leaseRenewalIntervalInSeconds    Eureka客户端向服务端发送心跳的时间间隔，单位为秒       30

* leaseExpirationDurationInSeconds   Eureka服务端在收到最后一次心跳之后等待的时间上限     90
                                 ，单位为秒。超过该时间之后服务端会将该服务实例从服务
                                 清单中剔除，从而禁止服务调用请求被发送到该示例上
                                
* nonSecurePort                   非安全的通信端口号                                    80

* securePort                      安全的通信端口号                                      443

* nonSecurePortEnabled            是否启用非安全的通信端口号                             true

* securePortEnabled               是否启用安全的通信端口号

* appname                         服务名，默认取spring.application.name的配置值

* hostname                        主机名，不配置的时候将根据操作系统的主机名来获取

```

注意：org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean类中，可以查看各个参数的默认值。

