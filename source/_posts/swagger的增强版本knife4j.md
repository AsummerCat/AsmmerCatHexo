---
title: swagger的增强版本knife4j
date: 2021-10-08 23:59:34
tags: [swagger]
---

# knife4j
也是可视化的接口文档

## 源码地址
```
Github
https://github.com/xiaoymin/swagger-bootstrap-ui
```
## 在线预览
```
http://knife4j.xiaominfo.com/doc.html

官方文档:
https://doc.xiaominfo.com/knife4j/documentation/get_start.html
```
<!--more-->
## 使用

### 基于swagger 仅仅更换前端UI
单纯皮肤增强
不使用增强功能,纯粹换一个swagger的前端皮肤，这种情况是最简单的,你项目结构下无需变更

可以直接引用swagger-bootstrap-ui的最后一个版本1.9.6或者使用knife4j-spring-ui

老版本引用
```
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>swagger-bootstrap-ui</artifactId>
  <version>1.9.6</version>
</dependency>
```
新版本引用
```
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-spring-ui</artifactId>
  <version>${lastVersion}</version>
</dependency>
```

### 基于SpringBoot直接引用
该包会引用所有的knife4j提供的资源，包括前端Ui的jar包
```
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-spring-boot-starter</artifactId>
  <version>${knife4j.version}</version>
</dependency>

```

### 基于SpringCloud引用
在Spring Cloud的微服务架构下,每个微服务其实并不需要引入前端的Ui资源,因此在每个微服务的Spring Boot项目下,引入knife4j提供的微服务starter
```
<dependency>
  <groupId>com.github.xiaoymin</groupId>
  <artifactId>knife4j-micro-spring-boot-starter</artifactId>
  <version>${knife4j.version}</version>
</dependency>
```
在网关聚合文档服务下,可以再把前端的ui资源引入
```
<dependency>
   <groupId>com.github.xiaoymin</groupId>
   <artifactId>knife4j-spring-boot-starter</artifactId>
   <version>${knife4j.version}</version>
</dependency>
```

## 创建配置


###  创建配置文件
```
package com.xxx.xxx.oauth.config;

import com.github.xiaoymin.knife4j.spring.annotations.EnableSwaggerBootstrapUi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @author cxc
 */
@Configuration
@EnableSwagger2
@EnableSwaggerBootstrapUi
public class Knife4jConfiguration {

    /**
     * 创建连接的包信息
     * <p>
     * 配置统一返回的controller路径RequestHandlerSelectors.basePackage
     *
     * @return 返回创建状况
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .useDefaultResponseMessages(false)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.syswin.systoon.platform.controller"))
                .paths(PathSelectors.any())
                .build();

    }


    /**
     * 设置文档信息主页的内容说明
     *
     * @return 文档信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Project textBook API ")
                .description("服务接口")
                .termsOfServiceUrl("http://localhost:8001/")
                .contact(new Contact("Mr Liu", "http://localhost:8999/", "liuyang@synway.cn"))
                .license("what")
                .version("1.0")
                .build();
    }

}
```
### 屏蔽生产环境显示文档
`application.yml` 配置文件中配置

```
knife4j:
  # 开启增强配置 
  enable: true
　# 开启生产环境屏蔽 true是关闭显示文档 false是显示文档
  production: true
```

### 开启密码验证
```

knife4j:
  # 开启增强配置 
  enable: true
　# 开启Swagger的Basic认证功能,默认是false
  basic:
      enable: true
      # Basic认证用户名
      username: test
      # Basic认证密码
      password: 123
```


## 注解说明
```
//标记模块 controller上
@Api(tags = "字典模块")


//标记接口上
 @ApiOperation(value = "删除字典", httpMethod = "POST")
    //单个参数
  @ApiImplicitParam(name = "id",value = "字典id",required = true,paramType = "header")
   //多个参数
   @ApiImplicitParams({
            @ApiImplicitParam(name = "activityId",value = "活动id",required = true),
            @ApiImplicitParam(name = "content",value = "评论内容",required = true)
    })
   
//标记实体上
 @ApiModel(description="字典表类")
 @ApiModelProperty(value = "字典描述")
```
