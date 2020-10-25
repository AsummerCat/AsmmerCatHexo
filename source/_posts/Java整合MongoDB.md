---
title: Java整合MongoDB
date: 2019-11-22 13:56:30
tags: [mongodb笔记]
---

# 方案一：Java直接整合Mongo

## 该方案需要使用mongo驱动

```java
 <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver</artifactId>
            <version>3.10.1</version>
        </dependency>

```

## 创建链接client&操作数据库

```java
 //数据库链接选项        
        MongoClientOptions mongoClientOptions = MongoClientOptions.builder().build();
 
        //数据库链接方式
        /** MongoDB 3.0以下版本使用该方法 */
        /*MongoCredential mongoCredential = MongoCredential.createMongoCRCredential("name", "dbname", "password".toCharArray());*/
 
        /** MongoDB 3.0及其以上版本使用该方法 */
        MongoCredential mongoCredential = MongoCredential.createScramSha1Credential("name", "dbname", "password".toCharArray());
 
        //数据库链接地址
        ServerAddress serverAddress = new ServerAddress("127.0.0.1", 27017);
 
        //获取数据库链接client
        MongoClient client= new MongoClient(serverAddress, mongoCredential, mongoClientOptions);
 
        //获取数据库对象
        DB db = client.getDB("dbname");
 
        //获取数据表对象
        DBCollection dbCollection = db.getCollection("collectionName");
 
        //数据库操作方法
        dbCollection.count();
```

<!--more-->

# 方案二：Spring Boot整合MongoDB（最简单）

## 添加spring-boot-mongo依赖

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
    
```

## 添加数据库配置（application.yml）：

```
spring:
  data:
    mongodb:
      username: username
      host: 127.0.0.1
      port: 27017
      password: password
      database: database
```

## 使用（在使用类中注入mongoTemple，实用类必须由Spring创建实例，否则无法注入）：

```
@Resource
    private MongoTemplate mongoTemplate;
```

*两个Spring boot版本测试：*

*2.0.5.RELEASE，不支持3.0以下的mongo版本*

*1.3.1.RELEASE，兼容所有的mongo版本*

# 方案三：Spring 整合MongoDB

## 添加必要依赖（版本不匹配会出现java.lang.ClassNotFoundException: org.springframework.core.SpringProperties错误）

```java
 <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.1.9.RELEASE</version><!-- 与spring-data-mongodb版本匹配 -->
        </dependency>
 
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongo-java-driver</artifactId>
            <version>2.13.0</version><!-- 与spring-data-mongodb版本匹配 -->
        </dependency>
        
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-mongodb</artifactId>
            <version>1.8.2.RELEASE</version><!-- 1.8.2.RELEASE版本支持所有的MongoDB版本 -->
        </dependency>
```

## 添加spring配置文件（applicationContext.xml）

```java
<?xml version="1.0" encoding="UTF-8"?>
 
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
 
    <import resource="classpath:mongo-spring.xml"/>
 
    <context:component-scan base-package="com.tools.*"/>
</beans>

```

## 添加mongo-spring配置文件（mongo-spring.xml）名字自定义，在applicationContext.xml中引入就行

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mongo="http://www.springframework.org/schema/data/mongo"
 
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                    http://www.springframework.org/schema/data/mongo
                    http://www.springframework.org/schema/data/mongo/spring-mongo-1.8.xsd">
 
    <bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
        <constructor-arg name="mongo" ref="mongo"/>
        <constructor-arg name="databaseName" value="dbname"/>
        <constructor-arg name="userCredentials" ref="mongoCredentials"/>
    </bean>
 
    <bean id="mongoCredentials" class="org.springframework.data.authentication.UserCredentials">
        <constructor-arg name="username" value="username" />
        <constructor-arg name="password" value="password" />
    </bean>
 
    <mongo:mongo host="127.0.0.1" port="27017">
        <mongo:options connections-per-host="10"
                       threads-allowed-to-block-for-connection-multiplier="100"
                       connect-timeout="1000" max-wait-time="1500" auto-connect-retry="true" />
    </mongo:mongo>
 
</beans>

```

## 连接数据库

```java
public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
 
        MongoTemplate mongoTemplate = (MongoTemplate)context.getBean("mongoTemplate");
        mongoTemplate.count(new Query(), "test");
    }
```

