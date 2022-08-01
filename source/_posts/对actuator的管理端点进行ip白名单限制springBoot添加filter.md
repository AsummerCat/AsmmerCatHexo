---
title: 对actuator的管理端点进行ip白名单限制springBoot添加filter
date: 2019-08-01 21:38:26
tags: [actuator,SpringCloud,SpringBoot]
---

在我们的SpringCloud应用中，我们会引入actuator来进行管理和监控我们的应用

常见的有：http://www.cnblogs.com/yangzhilong/p/8378152.html

如果开启 

```
endpoints.restart.enabled=true
```

则会有pause、restart等端点。

对shutdown、pause、restart等敏感指令我们需要进行一定的保护。当然actuator也考虑到了这点，对一些敏感的端点做了enable、sensitive以及security的校验。

为了使用方便，我们通常是如下的配置：

<!--more-->

```
# 禁用actuator管理端鉴权
management.security.enabled=false
# 启用shutdown   host:port/shutdown
endpoints.shutdown.enabled=true
# 禁用密码验证
endpoints.shutdown.sensitive=false
# 开启重启支持
endpoints.restart.enabled=true

# shutdown、pause、restart等的ip白名单地址
shutdown.whitelist=0:0:0:0:0:0:0:1,127.0.0.1,172.16.,10.18.
```

这么做的主要原因有：1、使用方便   2、方便集成到各种监控组建里去。

注：网上很多都是说的开启management的鉴权，类似如下（此方案会影响第三方监控组建的使用，不推荐使用）：

```
security.user.name=admin
security.user.password=admin
security.user.role=SUPERUSER

management.security.roles=SUPERUSER
```

如果不过这个security的交单会导致谁都可以直接post请求这些接口，故有了如下基于ip白名单的Filter方案：

ShutdownFilter.java

```
package com.mili.crm.eureka.filter;

import java.io.IOException;
import java.io.PrintWriter;
import java.util.Arrays;
import java.util.List;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;

import lombok.extern.slf4j.Slf4j;

/**
 * shutdown和pause的管理端点的ip白名单过滤
 *
 */
@WebFilter(filterName="shutdownFilter",urlPatterns= {"/shutdown","/pause","/restart"})
@Slf4j
@RefreshScope
public class ShutdownFilter implements Filter {
    @Value("${shutdown.whitelist:0:0:0:0:0:0:0:1}")
    private String[] shutdownIpWhitelist;
    
    @Override
    public void destroy() {
    }

    @Override
    public void doFilter(ServletRequest srequest, ServletResponse sresponse, FilterChain filterChain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) srequest;

        String ip = this.getIpAddress(request);
        log.info("访问shutdown的机器的原始IP：{}", ip);

        if (!isMatchWhiteList(ip)) {
            sresponse.setContentType("application/json");
            sresponse.setCharacterEncoding("UTF-8");
            PrintWriter writer = sresponse.getWriter();
            writer.write("{\"code\":401}");
            writer.flush();
            writer.close();
            return;
        }

        filterChain.doFilter(srequest, sresponse);
    }

    @Override
    public void init(FilterConfig arg0) throws ServletException {
        log.info("shutdown filter is init.....");
    }
    
    /**
     * 匹配是否是白名单
     * @param ip
     * @return
     */
    private boolean isMatchWhiteList(String ip) {
        List<String> list = Arrays.asList(shutdownIpWhitelist);
        return list.stream().anyMatch(item -> ip.startsWith(item));
    }
    
    /**
     * 获取用户真实IP地址，不使用request.getRemoteAddr();的原因是有可能用户使用了代理软件方式避免真实IP地址,
     * 可是，如果通过了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP值，究竟哪个才是真正的用户端的真实IP呢？
     * 答案是取X-Forwarded-For中第一个非unknown的有效IP字符串。
     * 
     * 如：X-Forwarded-For：192.168.1.110, 192.168.1.120, 192.168.1.130, 192.168.1.100
     * 
     * 用户真实IP为： 192.168.1.110
     * 
     * @param request
     * @return
     */
    private String getIpAddress(HttpServletRequest request) {
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("HTTP_CLIENT_IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```

然后在SpringBoot的启动类上加入如下注解

```
@ServletComponentScan("com.mili")
```

通过灵活配置这个白名单，就可以精准控制谁能访问了。