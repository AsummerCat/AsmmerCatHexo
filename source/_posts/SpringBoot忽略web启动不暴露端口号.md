---
title: SpringBoot忽略web启动不暴露端口号
date: 2020-10-16 15:40:25
tags: [SpringBoot]
---

在微服务中 如果入口只有一个的话 

像是dubbo 可以设置关闭对外暴露出服务的端口 那么只剩下 rpc的端口了

# 1.X版本 

```
public static void main(String[] args) throws InterruptedException {
        ApplicationContext ctx = new SpringApplicationBuilder().sources(启动类.class).web(false).run(args);
        logger.info("启动完成！");
    }
```

或者yml设置

```
#web_environment是否是web项目
spring.main.web_environment=true
#是否加载springboot banner
spring.main.show_banner=false
```



<!--more-->

# 2.X版本

NONE：非web  
REAVTIVE：reactive程序  
SERVLET：web项目     



```
  public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class)
            .web(WebApplicationType.NONE) // .REACTIVE, .SERVLET
            .bannerMode(Banner.Mode.OFF)
            .run(args);
    }
```

或者yml配置

```

#是否设定web应用，none-非web，servlet-web应用， reactive-reactive程序
spring.main.web-application-type=servlet
#加载springboot banner的方式：off-关闭，console-控制台，log-日志
spring.main.banner-mode=off
```

