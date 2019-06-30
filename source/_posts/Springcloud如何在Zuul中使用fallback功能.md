---
title: SpringCloud如何在Zuul中使用fallback功能zuul服务连接失败后的回退处理
date: 2019-06-30 23:37:34
tags: [springCloud,zuul]
---

# zuul服务连接失败后的回退处理

# Spring cloud 如何在Zuul中使用fallback功能

我们在项目中使用Spring cloud zuul的时候，有一种这样的需求，就是当我们的zuul进行路由分发时，如果后端服务没有启动，或者调用超时，这时候我们希望Zuul提供一种降级功能，而不是将异常暴露出来。



## * 自定义Zuul回退机制处理器*

需要实现 ZuulFallbackProvider 接口  1.x版本

需要实现 FallbackProvider 接口 2.x版本

<!--more-->

```java
package com.linjingc.zuulserver.fallbackProvider;

import com.alibaba.fastjson.JSONObject;
import org.springframework.cloud.netflix.zuul.filters.route.FallbackProvider;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.stereotype.Component;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * zuul 默认回滚处理
 *
 * @author cxc
 * @date 2019/6/30 23:23
 */
@Component
public class DefaultZuulFallbackProvider implements FallbackProvider {

    @Override
    public String getRoute() {
        //api服务id，如果需要所有调用都支持回退，则return "*"或return null
        return null;
    }

    /**
     * 如果请求用户服务失败，返回什么信息给消费者客户端
     */
    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {
            /**
             * 网关向api服务请求是失败了，但是消费者客户端向网关发起的请求是OK的，
             * 不应该把api的404,500等问题抛给客户端
             * 网关和api服务集群对于客户端来说是黑盒子
             */
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.OK.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return HttpStatus.OK.getReasonPhrase();
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                JSONObject r = new JSONObject();
                r.put("state", "9999");
                r.put("msg", "系统错误，请求失败");
                return new ByteArrayInputStream(r.toJSONString().getBytes("UTF-8"));
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                //和body中的内容编码一致，否则容易乱码
                headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
                return headers;
            }
        };
    }
}

```

