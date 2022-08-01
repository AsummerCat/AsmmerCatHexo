---
title: SpringColod异构服务整合node.js之sidecar
date: 2019-07-10 21:59:25
tags: [sidecar,SpringCloud]
---

# SpringColod异构服务整合node.js之sidecar

# 导入pom文件

```
  <!-- 异构系统模块 -->
        <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-netflix-sidecar</artifactId>
       </dependency>
```

<!--more-->

# 开启注解@EnableSidecar

```
@EnableSidecar
这个是个组合注解

集成异构微服务系统到 SpringCloud 生态圈中(比如集成 nodejs 微服务)。
 * 注意 EnableSidecar 注解能注册到 eureka 服务上，是因为该注解包含了 eureka 客户端的注解，该 EnableZuulProxy 是一个复合注解。
 * @EnableSidecar --> { @EnableCircuitBreaker、@EnableDiscoveryClient、@EnableZuulProxy } 包含了 eureka 客户端注解，同时也包含了 Hystrix 断路器模块注解，还包含了 zuul API网关模块。
@EnableCircuitBreaker
@EnableZuulProxy
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({SidecarConfiguration.class})
public @interface EnableSidecar {
}
```

# 配置文件修改

```
添加 didecar的相关配置
#异构微服务的配置， port 代表异构微服务的端口；health-uri 代表异构微服务的操作链接地址
sidecar:
  port: 8060
  health-uri: http://localhost:8060/health.json
```

# 新建node.js文件  node-service.js

```
var http = require('http');
var url = require('url');
var path = require('path');
 
//创建server
var server = http.createServer(function(req,res){
   //获取请求的路径
   var pathname = url.parse(req.url).pathname;
   res.writeHead(200, {'Content-Type' : 'application/json; charset=utf-8'});
   //访问http://localhost:8060/,将会返回{"index":"欢迎来到首页,该程序由Node.js提供"}
   if(pathname === '/'){
      res.end(JSON.stringify({"index" : "欢迎来到首页,该程序由Node.js提供"}));
   }
   //访问http://localhost:8060/health,将会返回{"status" : "UP"}
   else if (pathname === '/health.json'){
      res.end(JSON.stringify({"status" : "UP"}));
   }
   //其他情况返回404
   else{
      res.end("404")
   }
});
//创建监听，并打印日志
server.listen(8060,function(){
   console.log('listening on localhost:8060')
});

```

## 启动node服务

```
1.启动zureka
2.启动异构微服务
node node-service
3.启动zuul
访问异构微服务本身
http://localhost:8060/
通过zuul网关访问异构微服务
http://localhost:8040/sidecar-server
```

