---
title: gateway的各种路由配置断言工厂
date: 2019-10-08 16:34:15
tags: [gateway,springCloud,微服务]
---

# 断言工厂

[Spring Cloud](http://c.biancheng.net/spring_cloud/) Gateway 内置了许多路由断言工厂，可以通过配置的方式直接使用，也可以组合使用多个路由断言工厂。接下来为大家介绍几个常用的路由断言工厂类。

可以用来匹配不同的路由规则

## Path 路由断言工厂

Path 路由断言工厂接收一个参数，根据 Path 定义好的规则来判断访问的 URI 是否匹配。

```java
spring:
  cloud:
    gateway:
      routes:
        - id: host_route
    uri: http://c.biancheng.net
    predicates:
      - Path=/blog/detail/{segment}
```

  如果请求路径为 /blog/detail/xxx，则此路由将匹配。也可以使用正则，例如 /blog/detail/** 来匹配 /blog/detail/ 开头的多级 URI。

我们访问本地的网关：http://localhost：2001/blog/detail/36185 ，可以看到显示的是 http://c.biancheng.net/blog/detail/36185 对应的内容。  

<!--more-->

## Query 路由断言工厂

Query 路由断言工厂接收两个参数，一个必需的参数和一个可选的正则表达式。

```java
spring:
  cloud:
    gateway:
      routes:
        - id: query_route
      uri: http://c.biancheng.net
      predicates:
        - Query=foo, ba.
```

  如果请求包含一个值与 ba 匹配的 foo 查询参数，则此路由将匹配。bar 和 baz 也会匹配，因为第二个参数是正则表达式。

测试链接：`http://localhost：2001/?foo=baz。  `

## Method 路由断言工厂

Method 路由断言工厂接收一个参数，即要匹配的 HTTP 方法。

```
spring:
  cloud:
    gateway:
      routes:
        - id: method_route
  uri: http://baidu.com
  predicates:
    - Method=GET
```

## Header 路由断言工厂

Header 路由断言工厂接收两个参数，分别是请求头名称和正则表达式。

```java
spring:
  cloud:
    gateway:
      routes:
        - id: header_route
  uri: http://example.org
  predicates:
    - Header=X-Request-Id, \d+
```

  如果请求中带有请求头名为 x-request-id，其值与 \d+ 正则表达式匹配（值为一个或多个数字），则此路由匹配。

如果你想学习更多路由断言工厂的用法，可以参考官方文档进行学习。  

## 自定义路由断言工厂

  自定义路由断言工厂需要继承 AbstractRoutePredicateFactory 类，重写 apply 方法的逻辑。

在 apply 方法中可以通过 exchange.getRequest() 拿到 ServerHttpRequest 对象，从而可以获取到请求的参数、请求方式、请求头等信息。

apply 方法的参数是自定义的配置类，在使用的时候配置参数，在 apply 方法中直接获取使用。

命名需要以 RoutePredicateFactory 结尾，比如 CheckAuthRoutePredicateFactory，那么在使用的时候 CheckAuth 就是这个路由断言工厂的名称。代码如下所示。  

```java
@Component
public class CheckAuthRoutePredicateFactory
        extends AbstractRoutePredicateFactory<CheckAuthRoutePredicateFactory.Config> {
    public CheckAuthRoutePredicateFactory() {
        super(Config.class);
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange -> {
            System.err.println("进入了CheckAuthRoutePredicateFactory\t" + config.getName());
            if (config.getName().equals("zhangsan")) {
                return true;
            }
            return false;
        };
    }

    public static class Config {
        private String name;

        public void setName(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }
    }
}
```

使用示例如下所示：

```java
spring:
  cloud:
    gateway:
      routes:
  - id: customer_route
  uri: http://c.biancheng.net
  predicates:
    - name: CheckAuth
  args:
    name: zhangsan
```

