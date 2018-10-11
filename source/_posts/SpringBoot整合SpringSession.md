---
title: SpringBoot整合SpringSession
date: 2018-10-09 14:46:03
tags: [SpringBoot,SpringSession,redis]
---
>参考:  
>1.[spring-boot+spring-session集成](https://yq.aliyun.com/articles/182676?utm_content=m_29523)  
>2.[spring-session实现分布式session共享及自定义sessionid](https://blog.csdn.net/qq351790934/article/details/54930049)  
>3.[官方文档](http://projects.spring.io/spring-session/
http://docs.spring.io/spring-session/docs/current/reference/html5/guides/httpsession.html)
# 1.导入pom.xml

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
		
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
```
---

<!--more-->

# 2.引用redis

所以在配置文件中写入redis的配置文件

```
# Redis 配置(默认配置)
# Redis 数据库索引（默认为0）
spring.redis.database=0
# Redis 服务器地址
spring.redis.host=112.74.43.131
# Redis 服务器端口
spring.redis.port=6379
# Redis 服务器密码(默认为空)
spring.redis.password=xxxx
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1
# 设置连接超时
spring.redis.timeout=1000
logging.level.org.springframework.web=info
logging.level.com.linjingc.springbootandredis.controller=info

```

---

# 3.开启开启redis方式的session存储

两种方式 :
#### 3.1 配置文件

`spring.session.store-type=redis`

#### 3.2 注解

`@EnableRedisHttpSession`

---

# 4.扩展

虽然我们实现了redis方式存储的分布式session，但是在实际场景中可能还有一些需要优化的地方。

>一、修改cookies中sessionId的名字

>二、修改session存放在redis中的命名空间

>三、修改session超时时间


为什么要这样做呢，如果我们有两套不同系统A和B，cookies中session名称相同会导致同一个浏览器登录两个系统会造成相互的干扰，例如两个系统中我们存放在session中的用户信息的键值为user。默认情况下cookies中的session名称为JSESSIONID。当我们登录A系统后，在同一个浏览器下打开B系统，B系统B系统拿到的user实际上并非自己系统的user对象，这样会造成系统异常；而如果我们A、B系统存放用户信息的键值设置为不相同，例如userA和userB，当我们登录A系统后在登录B系统，我们打开浏览器调试模式方向两个系统只有一个sessionId，因为我们没有对二者的session名称以及存储的命名空间做区分，程序会认为就是自己可用的session。当我们推出B系统，注销session后。发现B系统的session也失效了，因为在redis中JSESSIONID对应的数据已经被设置过期了。


#### 4.1 针对命名空间，我们可以在配置文件上添加配置解决

1.配置文件:`spring.session.redis.namespace=xxxx`

2.注解:`@EnableRedisHttpSession(redisNamespace="xxxx")`

这样我们查看redis中就可以看到sping-session存储的key就变成了我们设置的值。

#### 4.2 针对超时时间，注解方式提供了响应的设置

`@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 3600)`


---

# 5.javaconfig方式自定义策略

我们可以采用javaconfig方式自定义策略来设置超时以及设置cookies名称，如下我们设置超时时间是1800秒，cookies名为MYSESSIONID

```
@Bean
public CookieHttpSessionStrategy cookieHttpSessionStrategy(){
    CookieHttpSessionStrategy strategy=new CookieHttpSessionStrategy();
    DefaultCookieSerializer cookieSerializer=new DefaultCookieSerializer();
    cookieSerializer.setCookieName("MYSESSIONID");//cookies名称
    cookieSerializer.setCookieMaxAge(1800);//过期时间(秒)
    strategy.setCookieSerializer(cookieSerializer);
    return strategy;
}
```

---

# 6.注意

 redis

`notify-keyspace-events`

默认情况下，这个功能是不开启的。

开启额外功能

通过下面的命令，来让你的Reids开启这个功能。

`redis-cli config set notify-keyspace-events Egx`

---

# 7. 测试方法

```
   @Value("${server.port}")
    String port;

    @RequestMapping(value = "/session", method = RequestMethod.GET)
    public Object getSession(HttpServletRequest request){
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("SessionId", request.getSession().getId());
        map.put("ServerPort", "服务端口号为 "+port);
        return map;
    }

同时启动两个相同的工程(比如：8080端口与9090端口)，访问 http://localhost:8080/session 与 http://localhost:9090/session 
我们可以得到以下结果：



{"SessionId":"01f353e1-5cd3-4fbd-a5d0-9a73e17dcec2","ServerPort":"服务端口号为 8080"}

{"SessionId":"01f353e1-5cd3-4fbd-a5d0-9a73e17dcec2","ServerPort":"服务端口号为 9090"}
```