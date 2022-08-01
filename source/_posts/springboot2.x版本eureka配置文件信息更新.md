---
layout: SpringBoot2.x
title: 版本eureka配置文件信息更新
date: 2019-07-10 21:57:33
tags: [SpringCloud,eureka,分布式配置中心]
---

# SpringBoot2.x版本eureka配置文件信息更新

# 获取ipaddress 修改

```
1.x` instance-id: ${spring.cloud.client.ipAddress}:${server.port}`

2.x `instance-id: ${spring.cloud.client.ip-address}:${server.port}`
```

<!--more-->

```
## eureka 客户端基本配置
eureka:
  instance:
    #路径
    prefer-ip-address: true
    #显示的Id
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
    # 每间隔30s，向服务端发送一次心跳，证明自己依然"存活"
    lease-renewal-interval-in-seconds: 15
    # 告诉服务端，如果我60s之内没有给你发心跳，就代表我"死"了，将我踢出掉。
    lease-expiration-duration-in-seconds: 30
  client:
    serviceUrl:
      defaultZone: http://admin:pass@localhost:8100/eureka
      healthcheck:
        enabled: true


```

默认配置:

```
 lease-renewal-interval-in-second =30
 lease-expiration-duration-in-seconds: 90
 这个基本上不推荐修改 用默认的就好了
```





