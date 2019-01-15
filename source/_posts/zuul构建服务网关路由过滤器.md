---
title: zuul构建服务网关路由过滤器三
date: 2019-01-15 15:46:22
tags: [SpringCloud,zuul]
---

# 服务网关可以过滤请求

`zuul.add-host-header=true`就能让原本有问题的重定向操作得到正确的处理。

## 基础

继承 ZuulFilter接口

<!--more-->

```java
package com.linjing.zuulserver;

import com.netflix.zuul.ZuulFilter;

/**
 * @author cxc
 * @date 2019/1/15 16:00
 */
public class TestFilter extends ZuulFilter {


    /**
     * 拦截类型
     * @return
     */
    @Override
    public String filterType() {
        
        return null;
    }

    /**
     * 拦截顺序 有多个的话 越小越前
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 过滤拦截 true拦截 
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return false;
    }

    /**
     * 拦截器具体事项
     * @return
     */
    @Override
    public Object run() {
        return null;
    }
}

```



## 案例

### 路由之前pre

```
package com.linjing.zuulserver;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class AccessFilter  extends ZuulFilter{

private static Logger logger= LoggerFactory.getLogger(AccessFilter.class);

    //filterType：过滤器的类型，它决定过滤器在请求的哪个生命周期中执行。这里定义为pre，代表会在请求被路由之前执行。
    //filterOrder：过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行。
    //shouldFilter：判断该过滤器是否需要被执行。这里我们直接返回了true，因此该过滤器对所有请求都会生效。实际运用中我们可以利用该函数来指定过滤器的有效范围。
    //run：过滤器的具体逻辑。这里我们通过ctx.setSendZuulResponse(false)令zuul过滤该请求，不对其进行路由，然后通过ctx.setResponseStatusCode(401)设置了其返回的错误码，当然我们也可以进一步优化我们的返回，
    //比如，通过ctx.setResponseBody(body)对返回body内容进行编辑等。

    @Override
    public String filterType() {
        //filterType：过滤器的类型，它决定过滤器在请求的哪个生命周期中执行。这里定义为pre，代表会在请求被路由之前执行。
        //return null;
      //  pre：路由之前
        //route：路由之时
        //post： 路由之后
        //error：发送错误调用
        return "pre";
    }

    @Override
    public int filterOrder() {
        //filterOrder：过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行。
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        //shouldFilter：判断该过滤器是否需要被执行。这里我们直接返回了true，
        //              因此该过滤器对所有请求都会生效。实际运用中我们可以利用该函数来指定过滤器的有效范围。

        return true;

    }

    @Override
    public Object run() {
        //run：过滤器的具体逻辑。这里我们通过ctx.setSendZuulResponse(false)令zuul过滤该请求，不对其进行路由
        //然后通过ctx.setResponseStatusCode(401)设置了其返回的错误码，当然我们也可以进一步优化我们的返回
        //比如，通过ctx.setResponseBody(body)对返回body内容进行编辑等。
        logger.info("启动了路由之前的过滤器了 1");

        RequestContext ctx=RequestContext.getCurrentContext();
        HttpServletRequest request=ctx.getRequest();
        logger.info("send {} request to {}",request.getMethod(),request.getRequestURI().toString());
        //
        //Object accessToken = request.getParameter("accessToken");
        //if(accessToken == null) {
        //    logger.warn("access token is empty");
        //    ctx.setSendZuulResponse(false);
        //    //ctx.setResponseStatusCode(401); //返回错误状态码
        //
        //    ctx.setResponseBody("该页面的accessToken 无法访问");
        //    logger.info(ctx.getResponseBody());
        //    return null;
        //}
        //logger.info("access token ok");


        return null;
    }
}
```



### 路由时route

```java
package com.linjing.zuulserver;

import com.netflix.zuul.ZuulFilter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
public class RouteFilter extends ZuulFilter{

private static Logger logger= LoggerFactory.getLogger(RouteFilter.class);


    @Override
    public String filterType() {
        return "route";
    }

    @Override
    public int filterOrder() {
        //filterOrder：过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行。
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        logger.info("启动了在路由请求时候被调用的过滤器了");
        return null;
    }
}
```

### 路由后post

```java
package com.linjing.zuulserver;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletResponse;

@Component
public class PostFilter extends ZuulFilter{

private static Logger logger= LoggerFactory.getLogger(PostFilter.class);


    @Override
    public String filterType() {
        return "post";
    }

    @Override
    public int filterOrder() {
        //filterOrder：过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行。
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        logger.info("启动了在在routing和error过滤器之后被调用的过滤器了");
        RequestContext ctx= RequestContext.getCurrentContext();
        //try{
        ctx.setResponseBody("post后置数据");
            doSomething();
        //}catch (Exception e){    //过滤器需要有严格的try()catch 进行处理
        //    ctx.set("error.status_code", HttpServletResponse.SC_INTERNAL_SERVER_ERROR);//错误编码
        //    ctx.set("error.exception", e);//错误对象
        //    ctx.set("error.message", e.getMessage());//错误信息
        //}


        return null;
    }



    private void doSomething() {
        throw new RuntimeException("错误信息...");
    }
}
```

### 发送错误调用error

```java
package com.linjing.zuulserver;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

@Component
public class ErrorFilter extends ZuulFilter{

private static Logger logger= LoggerFactory.getLogger(ErrorFilter.class);


    @Override
    public String filterType() {
      return   FilterConstants.ERROR_TYPE;
       // return "error";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        // only forward to errorPath if it hasn't been forwarded to already
        return true;

    }

    @Override
    public Object run() {
        logger.info("进入异常过滤器");
        RequestContext ctx = RequestContext.getCurrentContext();

        System.out.println(ctx.getResponseBody());

        ctx.setResponseBody("出现异常");

        return null;
    }
}
```



###  需要注意的是

1. 在定义过滤类型的时候 zuul有提供一个接口 可以直接使用 FilterConstants.ERROR_TYPE;

2. 获取当前req

3. ```java
   RequestContext ctx=RequestContext.getCurrentContext();
           HttpServletRequest request=ctx.getRequest();
   ```

4. ```java
   过滤器需要有严格的try()catch 进行处理
   ```

   3.配置属性`zuul.add-host-header=true`就能让原本有问题的重定向操作得到正确的处理。

# 构建一个异常后经过error返回的页面

继承 ErrorController

```java
package com.linjing.zuulserver;

import org.springframework.boot.autoconfigure.web.ErrorController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ErrorHandlerController implements ErrorController {

    /**
     * 出异常后进入该方法，交由下面的方法处理
     */
    @Override
    public String getErrorPath() {
        return "/error";
    }

    @RequestMapping("/error")
    public String error() {
        return "出现异常";
    }
}
```



# 配置文件

```java
server:
  port: 10010
spring:
  application:
    name: zuul-server

management:
  security:
    enabled: false

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8080/eureka/

#ribbon的读取 连接的超时时间设置
ribbon:
  ReadTimeout: 3000
  ConnectTimeout: 3000

zuul:
  host:
    connect-timeout-millis: 3000
    socket-timeout-millis: 3000

```

