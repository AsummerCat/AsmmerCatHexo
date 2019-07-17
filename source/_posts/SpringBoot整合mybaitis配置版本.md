---
title: SpringBoot整合mybaitis配置版本
date: 2019-07-16 17:13:41
tags: [springboot,mybaitis]
---

# SpringBoot整合mybaitis配置版本

demo地址:[config-demo](https://github.com/AsummerCat/mybatis-demo/tree/master/config-demo)

这种方式跟之前mvc中搭配mubatis的方式相差无几

这边数据库使用h2

# 详细代码

## 导入pom

```
 <!--mybaitis整合Springboot-->
 <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.1</version>
        </dependency>
```

<!--more-->

## 配置文件

```java
server:
  port: 8200

spring:
  datasource:
    hikari:
      driver-class-name: org.h2.Driver
      jdbc-url: jdbc:h2:~/test
    platform: h2
    #初始化脚本
    schema: classpath:db/schema.sql
      #在浏览器中开启控制台
  h2:
    console:
      enabled: true
      settings:
        web-allow-others: true
      path: /h2

mybatis:
  #mapping路径
  mapper-locations: classpath:mapping/*Mapper.xml
  #配置映射类所在的包名
  type-aliases-package: com.linjingc.configdemo.entity
  # 配置自动转换驼峰标识
  configuration:
    map-underscore-to-camel-case: true

```

## 在启动类添加包扫描 扫描mapper

@MapperScan("com.linjingc.configdemo.dao")

```java
/**
 * 配置篇
 */
@SpringBootApplication
//包扫描
@MapperScan("com.linjingc.configdemo.dao")
public class ConfigDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigDemoApplication.class, args);
    }

}
```

## controller Server  entity常规写法

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Resource
    private UserService userService;

    @RequestMapping("/{name}")
    public BasicUser findUser(@PathVariable String name) {
        return userService.findUser(name);
    }

    @RequestMapping("/save")
    @ResponseBody
    public String save() {
        userService.saveUser();
        return "保存成功";
    }
}
```

```java
/**
 * @author cxc
 */
@Service
public class UserService {
    @Resource
    private UserMapper userMapper;

    public BasicUser findUser(String name) {
        return userMapper.findUser(name);
    }

    public void saveUser() {
        for (int i = 0; i < 100; i++) {
            userMapper.insertUser("小明" + i, "123456", i);
        }
    }
}
```

```java
/**
 * 用户类
 *
 * @author cxc
 */
@Data
@ToString
public class BasicUser {
    private Long id;
    private String name;
    private Integer age;
    private String textNode;
}

```

## 创建Mapper接口

添加 @Repository注解

```java
@Repository
public interface UserMapper {
    BasicUser findUser(@Param("name") String name);

    int insertUser(@Param("name") String name, @Param("password") String password, @Param("age") Integer age);

}
```

这里需要注意

- ```java
  #{name} 和${name}的区别    #{}代表自动拼接``  ${}表示需要手动添加``
  ```

  - ```
    这是一个很容易忽视的点，记住：接口名与Mybatis的映射文件名一定要一模一样
    
    ```

## 接着维护映射xml

在resource中创建个 文件夹 mapping 

这里就是 mapper-locations: classpath:mapping/*Mapper.xml 维护的

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.linjingc.configdemo.dao.UserMapper">


    <select id="findUser" resultType="com.linjingc.configdemo.entity.BasicUser">
        select * from user where name = #{name}
    </select>

    <insert id="insertUser">
        INSERT INTO USER(NAME, PASSWORD, AGE) VALUES(#{name}, #{password}, #{age})
    </insert>
</mapper>

```



