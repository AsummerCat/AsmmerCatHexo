---
title: 多线程中spring注入失败
date: 2018-11-28 13:40:04
tags: [多线程,Spring]
---

# 多线程中spring注入失败

导致需要注入的service 变成null


* 问题：多线程中无法共享主线程中的bean。

* 解决：我们手动获取bean 
<!--more-->

web容器在启动应用时，spring容器是无法感知多线程的那些bean的，所以多线程的bean类无法获取spring容器的上下文，并不能通过@Autowired注入需要的bean



# 解决 

## 创建一个工具类手动获取bean

```
package com.biniu.heben.utils.thread;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import java.util.Locale;
import java.util.Map;

/**
 * @author cxc
 */
@Component
public class SpringContextUtil implements ApplicationContextAware {

    private static ApplicationContext applicationContext = null;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {
        applicationContext = context;
    }


    /**
     * 获取applicationContext对象
     *
     * @return
     */
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }


    /**
     * 根据bean的id来查找对象
     *
     * @param id
     * @return
     */

    public static <T> T getBeanById(String id) {
        return (T) applicationContext.getBean(id);
    }


    /**
     * 根据bean的class来查找对象
     *
     * @param c
     * @return
     */
    public static <T> T getBeanByClass(Class c) {
        return (T) applicationContext.getBean(c);
    }


    /**
     * 根据bean的class来查找所有的对象(包括子类)
     *
     * @param c
     * @return
     */
    public static Map getBeansByClass(Class c) {
        return applicationContext.getBeansOfType(c);
    }

    public static String getMessage(String key) {
        return applicationContext.getMessage(key, null, Locale.getDefault());
    }

}
```


## 线程池

```
package com.biniu.heben.utils.thread;

import org.springframework.stereotype.Component;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 线程池
 *
 * @author cxc
 * @date 2018/11/28 10:58
 */
@Component
public class ThreadPoolNewCache {
    ExecutorService executorService = Executors.newSingleThreadExecutor();

    public void runBatchUpdateNextNum() {
        executorService.submit(new MyRunnableUpdateNextNum());
        executorService.shutdown();
    }

}

```

## 线程实现类

```
package com.biniu.heben.utils.thread;

import com.biniu.heben.service.UserService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * @author cxc
 * @date 2018/10/29 17:31
 * 缓存线程池 的跑批更新上级的下一级数量
 */
@Component
public class MyRunnableUpdateNextNum implements Runnable {
    private static Logger logger = LoggerFactory.getLogger(MyRunnableUpdateNextNum.class);


    @Override
    public void run() {
        logger.info("-->>>>线程池 跑批更新上级的下一级数量" + LocalDateTime.now() + "开始");
        try {
            UserService userService = (UserService) SpringContextUtil.getBeanByClass(UserService.class);
            userService.batchUpdateNextNum();
        } catch (Exception e) {
            logger.error("跑批失败{}", e);
        }
        logger.info("-->>>>线程池 跑批更新上级的下一级数量" + LocalDateTime.now() + "结束");
    }
}
```

这里需要注意的是 使用了

```
UserService userService = (UserService) SpringContextUtil.getBeanByClass(UserService.class);

手动获取bean
```

