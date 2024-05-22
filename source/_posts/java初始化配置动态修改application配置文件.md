---
title: java初始化配置动态修改application配置文件
date: 2024-05-22 23:30:35
tags: [java, springboot]
---
# java初始化配置动态修改application配置文件
使用场景: application.properties写死一些配置 ,我需要在项目启动的时候进行替换,不想修改Bean相关信息去手动注入

`直接修改 系统读取application.properties 进行替换后 启动程序`

<!--more-->

## 第一步 创建读取配置文件的后置处理器

`CustomEnvironmentPostProcessor`

```

import org.springframework.boot.SpringApplication;
import org.springframework.boot.env.EnvironmentPostProcessor;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.MapPropertySource;

import java.util.HashMap;
import java.util.Map;


public class CustomEnvironmentPostProcessor implements EnvironmentPostProcessor {

package com.goal.env;

import com.tools.util.FunctionUtil;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.env.EnvironmentPostProcessor;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.MapPropertySource;

import java.util.HashMap;
import java.util.Map;

/**
 * @ClassName: 自定义环境变量处理器
 * @author cxc
 */
public class CustomEnvironmentPostProcessor implements EnvironmentPostProcessor {

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        System.out.println("-------------------------------------------------------------");
        System.out.println("INIT CustomEnvironmentPostProcessor postProcessEnvironment");
        System.out.println("-------------------------------------------------------------");
        String jdbcType = environment.getProperty("jdbc_type");
        if(null==jdbcType||jdbcType.isEmpty()){
            throw new RuntimeException("无法识别数据库对应类型,请检查配置文件: jdbc_type");
        }
        System.out.println("JDBC_TYPE:"+ jdbcType);
        assert jdbcType != null;
        String dynamicDialect = FunctionUtil.getDialect(jdbcType);
        if(null==dynamicDialect||dynamicDialect.isEmpty()){
         throw new RuntimeException("对应数据库无法找到对应方言:"+jdbcType);
        }
        // 设置新的属性值
        Map<String, Object> customProperties = new HashMap<>();
        System.out.println("DB dialect INIT :"+ dynamicDialect);
        customProperties.put("spring.jpa.properties.hibernate.dialect", dynamicDialect);
        customProperties.put("spring.jpa.database-platform", dynamicDialect);
        environment.getPropertySources().addFirst(new MapPropertySource("customPropertySource", customProperties));
        System.out.println("-------------------------------------------------------------");

    }

}
```



## 第二步 添加spring.factories

在resources中新增`spring.factories`

用来替换系统的启动变量
```
org.springframework.boot.env.EnvironmentPostProcessor=com.xx.xx.env.CustomEnvironmentPostProcessor
```

