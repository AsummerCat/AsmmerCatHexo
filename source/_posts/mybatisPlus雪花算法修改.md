---
title: mybatisPlus雪花算法修改
date: 2023-07-25 15:47:51
tags: [mybatis]
---

# mybatisPlus雪花算法修改

线上可能会出现这么个情况

一个服务发布了很多个系统

代码中使用默认的`mubatis-plus`集成的雪花算法生成主键

导致了高并发情况下,生成的主键重复

生成原因:
```
雪花算法的:
workerId和datacenterId 重复了

在K8S环境下同一个服务部署多台可能导致生成的
workerId和datacenterId重复获取本地MAC地址相关内容
```
<!--more-->

修复方案:
使生成的workerId和datacenterId不重复,

1.可集成 开源框架使用zk或者redis生成workerId和datacenterId

2.直接代码启动类上修改成随机数 避免重复

配置文件添加
```
mybatis-plus:
  global-config:
    worker-id: ${random.int(1,31)}
    datacenter-id: ${random.int(1,31)}    
```
启动类修改
在1-31之间随机 worker-id 和 datacenter-id

```
package com.xxx;

import com.baomidou.mybatisplus.core.toolkit.IdWorker;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;


public class Application {


    public static void main(String[] args) throws IllegalAccessException {
//		SpringApplication.run(Application.class, args);
        SpringApplication app = new SpringApplication(Application.class);
        app.run(args);
        log.info("启动成功");
        log.info("Application is success!");
        log.info("----------------------------------");
        IdWorker.initSequence(workerId, datacenterId);
        log.info("-------雪花算法workerId:{}---------", workerId);
        log.info("-------雪花算法datacenterId:{}-----", datacenterId);
        log.info("----------------------------------");

    }

    private static Long workerId;
    private static Long datacenterId;

    @Value("${mybatis-plus.global-config.worker-id}")
    public void setWorkerId(Long accessKeyId) {
        CygnusPortalApplication.workerId = accessKeyId;
    }

    @Value("${mybatis-plus.global-config.datacenter-id}")
    public void setDatacenterId(Long secret) {
        CygnusPortalApplication.datacenterId = secret;
    }


}

```