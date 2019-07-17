---
title: SpringBoot整合mybaitis注解版本
date: 2019-07-16 17:15:33
tags: [springboot,mybaitis]
---

# SpringBoot整合mybaitis注解版本

demo地址:[note-demo](https://github.com/AsummerCat/mybatis-demo/tree/master/note-demo)

# 详细代码

## 导入pom

```java
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
  port: 8100

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

# 指定POJO扫描包来让mybatis自动扫描到自定义POJO
mybatis:
  type-aliases-package: com.linjingc.notedemo.entity
  # 配置自动转换驼峰标识
  configuration:
    map-underscore-to-camel-case: true
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
            userMapper.insert("小明" + i, "123456", i);
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

添加 @Mapper注解 

```
/**
 * @author cxc
 */
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM USER WHERE NAME = #{name}")
    BasicUser findUser(String name);

    @Insert("INSERT INTO USER(NAME, PASSWORD, AGE) VALUES(#{name}, #{password}, #{age})")
    int insert(@Param("name") String name, @Param("password") String password, @Param("age") Integer age);
}
```

这样就基本实现了

