---
title: SpringBoot整合Pagehelper插件实现分页功能
date: 2019-07-16 17:16:32
tags: [SpringBoot,mybaitis]
---

# SpringBoot整合Pagehelper插件实现分页功能

这边的话 mybaitis使用的注解的

demo地址:[pagehelper-demo](https://github.com/AsummerCat/mybatis-demo/tree/master/pagehelper-demo)

# 首先导入pom

```java
   <!--pagehelper-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.12</version>
        </dependency>
```

<!--more-->

# 创建分页插件配置文件

```java
/**
 * 分页插件配置文件
 * 将该类加到spring容器里
 * pageHelper
 * @Link https://pagehelper.github.io/docs/howtouse/ 官网文档地址
 */
@Configuration
public class PageHelperConfig {
    @Bean
    public PageHelper pageHelper() {
        PageHelper pageHelper = new PageHelper();
        Properties properties = new Properties();

        properties.setProperty("offsetAsPageNum", "true");
        properties.setProperty("rowBoundsWithCount", "true");
        properties.setProperty("reasonable", "true");
        properties.setProperty("dialect", "h2");
        pageHelper.setProperties(properties);
        return pageHelper;
    }
}
```

# 配置文件

```java
server:
  port: 8500

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

# controller Server  entity常规写法

```java
  /**
     * 分页查询
     *
     * @param page     页数
     * @param pagesize 单页数量
     * @return
     */
    @RequestMapping("/pageFindUser/{page}/{pagesize}")
    @ResponseBody
    public String pageFindUser(@PathVariable Integer page, @PathVariable Integer pagesize) {
        List<BasicUser> all = userService.getAll(page, pagesize);
        return all.toString();
    }

```

```java
package com.linjingc.pagehelperdemo.service;

import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;
import com.linjingc.pagehelperdemo.dao.UserMapper;
import com.linjingc.pagehelperdemo.entity.BasicUser;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import java.util.List;

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

    /**
     * PageHelper分页查询全部
     *
     * @param page
     * @param pageSize
     */
    public List<BasicUser> getAll(int page, int pageSize) {
        //这里就是分页了
        PageHelper.startPage(page, pageSize);

        PageInfo<BasicUser> userPageInfo = new PageInfo<>(userMapper.getAll());

        List<BasicUser> list = userPageInfo.getList();

        return list;
    }
}
```

# Pagehelper插件

实现分页特别简单

只要查询前加入PageHelper.startPage(page, pageSize); 然后包裹住你需要查询的方法

```
  //这里就是分页了
        PageHelper.startPage(page, pageSize);

        PageInfo<BasicUser> userPageInfo = new PageInfo<>(userMapper.getAll());

        List<BasicUser> list = userPageInfo.getList();
```

