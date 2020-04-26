---
title: SpringCloudAlibaba-nacos注册中心config
date: 2020-04-26 10:52:48
tags: [SpringCloudAlibaba,nacos]
---

# SpringCloudAlibaba-nacos注册中心config

nacos 可以用来做注册中心 和 动态config参数

# [demo地址](https://github.com/AsummerCat/nacos-demo)

# SpringBoot使用动态配置

##  [官方文档](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)

## 1.导入jar

```java
<dependency>
    <groupId>com.alibaba.boot</groupId>
    <artifactId>nacos-config-spring-boot-starter</artifactId>
    <version>${latest.version}</version>
</dependency>
版本 0.2.x.RELEASE 对应的是 Spring Boot 2.x 版本，版本 0.1.x.RELEASE 对应的是 Spring Boot 1.x 版本。
```

## 2.在 `application.properties` 中配置 Nacos server 的地址：

```
nacos.config.server-addr=127.0.0.1:8848
```

## 3.通过 Nacos 的 `@NacosValue` 注解设置属性值。

```java
@Controller
@RequestMapping("config")
public class ConfigController {

    @NacosValue(value = "${useLocalCache:false}", autoRefreshed = true)
    private boolean useLocalCache;

    @RequestMapping(value = "/get", method = GET)
    @ResponseBody
    public boolean get() {
        return useLocalCache;
    }
}
```

<!--more-->

# Spring cloud使用动态配置

## 1.导入nacos的starter包

```java
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>${latest.version}</version>
</dependency>
    注意：版本 2.1.x.RELEASE 对应的是 Spring Boot 2.1.x 版本。版本 2.0.x.RELEASE 对应的是 Spring Boot 2.0.x 版本，版本 1.5.x.RELEASE 对应的是 Spring Boot 1.5.x 版本。
```

## 2.在 `bootstrap.properties` 中配置 Nacos server 的地址和应用名

```
  cloud:
    nacos:
      ## 注册中心地址
      server-addr: 127.0.0.1:8848
      discovery:
        ## 分组
        group: test
```

## 3.dataIdd的默认格式

```
${prefix}-${spring.profile.active}.${file-extension}
1.prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
2.spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 3.spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}
4.file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。
```

### 4.通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新：

这个注解 需要放置在你添加@value的类上 放在启动类不生效

```
@RestController
@RequestMapping("/config")
@RefreshScope
public class ConfigController {

    @Value("${useLocalCache:false}")
    private boolean useLocalCache;

    @RequestMapping("/get")
    public boolean get() {
        return useLocalCache;
    }
}
```

# 动态配置的写法 

必须使用 bootstrap.properties 配置文件来配置Nacos Server 地址

当动态配置刷新时，会更新到 Enviroment中，因此这里每隔一秒中从Enviroment中获取配置

你可以通过配置 `spring.cloud.nacos.config.refresh.enabled=false` 来关闭动态刷新

## nacos中的写法

![](/img/2020-04-26/nacos-config.png)

注意的是dataId里面默认是 spring.application.name-环境.Properties

## 客户端写法

### 导入pom.xml

```
     <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```

### 编写@value注解

```java
	@Value("${hello.flag:默认值}")
	private String test;
```

### 开启自动刷新

注意的是:这个注解需要放在 你标注@value的那个类上

```
@RefreshScope
```

### 配置bootstartp.Properties

```java
spring:
  profiles:
    active: dev
  cloud:
    nacos:
      ## 注册中心地址
      server-addr: 127.0.0.1:8848
      discovery:
        ## 分组
          group: test
      config:
        # config默认是跟注册注册中心的路径一样 也可以单独配置
        server-addr: 127.0.0.1:8848
         # 这里是调用以什么后缀的格式
        file-extension: yaml
         # 这里就是config分组 (和服务分组的概念不一样)
        group: test
        
     以这个形式会调用 nacos-demo-dev.yaml 的配置文件   
```

## 这样一件简单的config就实现了

## 单项目使用多个配置文件

```
## 三种方式优先级方式
    ## 默认规则  >  ext-config[n]  >  shared-dataids
```

```java
spring:
  profiles:
    active: dev
  cloud:
    nacos:
      ## 注册中心地址
      server-addr: 127.0.0.1:8848
      discovery:
        ## 分组
          group: test
      config:
        file-extension: yaml
        group: test
        ## 共享配置 方式1
        shared-configs: common.yaml,redis.yaml
        ## 配置自动刷新
        refreshable-dataids: common.yaml,redis.yaml

        ## 共享配置 方式2
        extension-configs:
          - data-id: greeting.yml
            group: common
            refresh: true
          - data-id: author.yml
            group: common
            refresh: true

## 三种方式优先级方式
    ## 默认规则  >  ext-config[n]  >  shared-dataids
```

### 常用方式 是 extension-configs 实现共享配置

第一种的话是没办法指定分组的

```java
 这样可以实现指定分组
 extension-configs:
          - data-id: greeting.yml
            group: common
            refresh: true
          - data-id: author.yml
            group: common
            refresh: true
```

## 最后还有一点Namespace的概念 可以选择某个命名空间下的配置文件 比如A空间下的dev配置 

目的是为了 多个项目组可以用不同的配置空间 不互相影响

默认分组是 public

`提示也是必须放入bootstrap.properties 文件中`

![](/img/2020-04-26/nacos-namespace.png)

### yml配置

```
cloud:
    nacos:
      config:
        server-addr: 192.168.92.1:8848 # Nacos 地址与端口
        namespace: e5fc372c-ad66-4e0e-a353-a217d0a315ba # 指定命名空间ID 从这里获取配置
```

##  (重点)还有种方式 在启动时候传入namespaceId 动态选择

```java
注意：不论用哪一种方式实现。对于指定环境的配置（spring.profiles.active=DEV、spring.cloud.nacos.config.group=DEV_GROUP、spring.cloud.nacos.config.namespace=83eed625-d166-4619-b923-93df2088883a），都不要配置在应用的bootstrap.properties中。而是在发布脚本的启动命令中，用-Dspring.profiles.active=DEV的方式来动态指定，会更加灵活！。
```





# 服务注册的使用

spring.cloud.nacos.server-addr 

```JAVA
spring:
  profiles:
    active: dev
  cloud:
    nacos:
      ## 注册中心地址
      server-addr: 127.0.0.1:8848
      discovery:
        ## 分组 默认分组为DEAFULT-GROUP
          group: test
```

# nacos持久化

## 单机模式支持mysql

在0.7版本之前，在单机模式时nacos使用嵌入式数据库实现数据的存储，不方便观察数据存储的基本情况。0.7版本增加了支持mysql数据源能力，具体的操作步骤：

- 1.安装数据库，版本要求：5.6.5+
- 2.初始化mysql数据库，数据库初始化文件：nacos-mysql.sql [地址](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)
- 3.修改conf/application.properties文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://11.162.196.16:3306/nacos_devtest?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=nacos_devtest
db.password=youdontknow
```

再以单机模式启动nacos，nacos所有写嵌入式数据库的数据都写到了mysql

## 集群模式运行

3个或3个以上Nacos节点才能构成集群。

```
因此开源的时候推荐用户把所有服务列表放到一个vip下面，然后挂到一个域名下面

http://ip1:port/openAPI 直连ip模式，机器挂则需要修改ip才可以使用。

http://VIP:port/openAPI 挂载VIP模式，直连vip即可，下面挂server真实ip，可读性不好。

http://nacos.com:port/openAPI 域名 + VIP模式，可读性好，而且换ip方便，推荐模式
```

![集群模式](/img/2020-04-26/nacos-cluster.jpeg)

### 修改配置集群配置文件

在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，请每行配置成ip:port。（请配置3个或3个以上节点）

```
# ip:port
200.8.9.16:8848
200.8.9.17:8848
200.8.9.18:8848
```

### 配置 MySQL 数据库 [地址](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)

```
生产使用建议至少主备模式，或者采用高可用数据库。
```

### 客户端配置

```
这边的话 跟普通的一样
集群模式下 可以配置个niginx 反向代理出地址 给微服务
```

### 启动

```
Windows单机启动直接双击 startup.cmd，Linux执行 ./startup.sh -m stanalone

配置好集群后，Windows集群启动CMD窗口下执行 startup.cmd -m cluster，Linux直接执行./startup.sh
```

## [ 配置文件地址](https://github.com/alibaba/nacos/blob/master/distribution/conf/application.properties)

# Nacos 监控手册

可以使用第三方监控平台

[文档地址](https://nacos.io/zh-cn/docs/monitor-guide.html)