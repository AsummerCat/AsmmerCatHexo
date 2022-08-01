---
title: java使用拦截器统计每个http请求的响应时间
date: 2019-09-26 15:03:06
tags: [java,工具类,SpringMvc]
---

# java使用拦截器统计每个http请求的响应时间

# 创建拦截器

```
package com.example.demo;

import lombok.extern.log4j.Log4j2;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Http请求时间统计
 * 拦截所有请求
 */
@Log4j2
public class HttpRquestTimeInterceptor extends HandlerInterceptorAdapter {

    //这里还可以优化一下 记录指标啥的 客户端ip之类

    ThreadLocal<Long> localThread = new ThreadLocal<Long>();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

         //线程方式
        localThread.set(System.currentTimeMillis());
        //request方式
        request.setAttribute("_startTime", System.currentTimeMillis());

        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        Long endTime = System.currentTimeMillis();
        Long startTime = (Long) request.getAttribute("_startTime");
        System.out.println(request.getServletPath() + " >> http请求结束:" + (endTime - startTime)+"毫秒");

    }
}
```



<!--more-->

# 拦截器注入

```
package com.example.demo;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * 和springmvc的webmvc拦截配置一样
 *
 * @author BIANP
 */

@Configuration
public class WebAppConfigurer implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 可添加多个
        registry.addInterceptor(new HttpRquestTimeInterceptor()).addPathPatterns("/**");
    }

}
```

