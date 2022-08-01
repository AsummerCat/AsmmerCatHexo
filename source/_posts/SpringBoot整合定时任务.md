---
title: SpringBoot整合定时任务
date: 2019-01-11 16:09:15
tags: [SpringBoot]
---

# [demo地址](https://github.com/AsummerCat/springBootAndTasks)

# SpringBoot整合定时任务

 利用 注解 和 Scheduling Tasks 完成定时任务的编写

<!--more-->

推荐一个在线Cron表达式生成的网站

[在线Cron表达式](http://cron.qqe2.com/)

# 首先照旧导入pom



```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-quartz</artifactId>
		</dependency>
```



# 启动类添加开启定时任务的注解

在它的程序入口加上@EnableScheduling,开启调度任务。

```
package com.linjing.tasks;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
//开启定时任务调度
@EnableScheduling
public class TasksApplication {
    public static void main(String[] args) {
        SpringApplication.run(TasksApplication.class, args);
    }
}
```

# 创建定时任务

创建一个定时任务，每过5s在控制台打印当前时间。

通过在方法上加@Scheduled注解，表明该方法是一个调度任务。

@Scheduled(fixedRate = 5000) ：上一次开始执行时间点之后5秒再执行
@Scheduled(fixedDelay = 5000) ：上一次执行完毕时间点之后5秒再执行
@Scheduled(initialDelay=1000, fixedRate=5000) ：第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次@Scheduled(cron=" /5 ") ：通过cron表达式定义规则，什么是cro表达式，自行搜索引擎。

```
package com.linjing.tasks;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.time.LocalTime;

/**
 * @author cxc
 * @date 2019/1/11 16:27
 */
@Component
public class TestTasks {

    @Scheduled(fixedRate = 5000)
    public void test() {
        System.out.println("普通定时任务 5秒执行一次当前时间为:" + LocalTime.now());
    }

    @Scheduled(cron = "0/2 * * * * ?")
    public void cronTest() {
        System.out.println("Cron表达式 每2秒执行一次 当前时间为:" + LocalTime.now());
    }
}

```

一个简单的定时任务就开启了 每间隔5秒执行一次



# 测试

```
Cron表达式 每2秒执行一次 当前时间为:16:36:26.004
Cron表达式 每2秒执行一次 当前时间为:16:36:28.005
普通定时任务 5秒执行一次当前时间为:16:36:29.550
Cron表达式 每2秒执行一次 当前时间为:16:36:30.002
Cron表达式 每2秒执行一次 当前时间为:16:36:32.005
Cron表达式 每2秒执行一次 当前时间为:16:36:34.004
普通定时任务 5秒执行一次当前时间为:16:36:34.548
```



# 完成