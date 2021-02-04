---
title: 携程Apollo阿波罗的安装(二)
date: 2021-02-04 10:25:06
tags: [apollo,分布式配置中心]
---

[官网文档](https://github.com/ctripcorp/apollo/wiki/Quick-Start)
[demo地址](https://github.com/AsummerCat/apollodemo)

# 服务端安装
因为是根据数据库来进行操作的 需要导入脚本
```
创建两个数据库分别导入相应的脚本数据
注意: 导入脚本 会清空原有数据
数据库名称: ApolloPortalDB  ApolloConfigDB
```

<!--more-->

## 配置数据库信息
```
Apollo服务端需要知道如何连接到你前面创建的数据库，所以需要编辑demo.sh，修改ApolloPortalDB和ApolloConfigDB相关的数据库连接串信息。

注意：填入的用户需要具备对ApolloPortalDB和ApolloConfigDB数据的读写权限。

#apollo config db info
apollo_config_db_url=jdbc:mysql://localhost:3306/ApolloConfigDB?characterEncoding=utf8
apollo_config_db_username=用户名
apollo_config_db_password=密码（如果没有密码，留空即可）

# apollo portal db info
apollo_portal_db_url=jdbc:mysql://localhost:3306/ApolloPortalDB?characterEncoding=utf8
apollo_portal_db_username=用户名
apollo_portal_db_password=密码（如果没有密码，留空即可）
```

## 启动服务端
```
 确保端口未被占用
Quick Start脚本会在本地启动3个服务，分别使用8070, 8080, 8090端口，请确保这3个端口当前没有被使用。

例如，在Linux/Mac下，可以通过如下命令检查：

lsof -i:8080
3.2 执行启动脚本
./demo.sh start
当看到如下输出后，就说明启动成功了！

==== starting service ====
Service logging file is ./service/apollo-service.log
Started [10768]
Waiting for config service startup.......
Config service started. You may visit http://localhost:8080 for service status now!
Waiting for admin service startup....
Admin service started
==== starting portal ====
Portal logging file is ./portal/apollo-portal.log
Started [10846]
Waiting for portal startup......
Portal started. You can visit http://localhost:8070 now!
```

访问8070即可进入可视化界面

# 客户端使用
注意服务器是使用eureka的
## 导入pom.xml
```
        <dependency>
            <groupId>com.ctrip.framework.apollo</groupId>
            <artifactId>apollo-client</artifactId>
            <version>1.7.0</version>
        </dependency>
        
      <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-context</artifactId>
        </dependency>
        
```
## 创建app.properties
在`resources下创建一个META-INF文件夹再创建app.properties`
内容:
```
#自定义的appid名称，区分不同的应用
app.id=SampleApp
```
## 修改application.yml
添加apollo的相关配置
```
apollo:
  #eureka配置中心地址
  meta: http://localhost:8080
  cacheDir: /opt/data/test
  bootstrap:
    enabled: true
    eagerLoad:
      enabled: true

```

## 代码中开启apollo注解

## 使用 
就跟spring cloud config一样的使用
```
@value(#{xxx属性:默认值})

例如:@Value("${timeout:100}")
```