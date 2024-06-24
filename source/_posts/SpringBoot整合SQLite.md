---
title: SpringBoot整合SQLite
date: 2024-06-24 22:59:30
tags: [SpringBoot,SQLite]
---
# &#x20;1.SQLite安装

直接本地创建一个xxx.db的文件就可以了 默认里面是没有表的

## 1.1SQLite密码配置

这里使用SQLCipher进行给sqlLite进行加密

    创建或打开数据库：使用 SQLCipher 打开数据库时，您需要提供一个密钥来加密数据库或解密已加密的数据库

    一旦密钥被设置，所有的数据库操作都会自动使用这个密钥进行加密和解密。

    -- 设置加密密钥
    PRAGMA key = 'password';

<!--more-->
## 2.获取pom依赖整合SQLite

    <!-- SQLite JDBC Driver -->
    <dependency>
        <groupId>org.xerial</groupId>
        <artifactId>sqlite-jdbc</artifactId>
        <version>3.34.0</version> <!-- Use the latest version -->
    </dependency>
     
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

## 3.application.properties

    spring.datasource.driver-class-name=org.sqlite.JDBC
    #绝对位置配置方式
    #spring.datasource.url=jdbc:sqlite:E:/db/test.db
    #相对位置配置方式
    spring.datasource.url=jdbc:sqlite::resource:db/test.db


    如果使用JPA的话:
    spring.datasource.url=jdbc:sqlite:tutorial.db
    spring.datasource.driver-class-name=org.sqlite.JDBC
    spring.datasource.hikari.maximum-pool-size=1
    spring.jpa.properties.hibernate.dialect=org.sqlite.hibernate.dialect.SQLiteDialect
    spring.jpa.show-sql=true
    spring.jpa.hibernate.ddl-auto=update

## 4.测试代码

```
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
            // 1、建表 DDL
        String createUser = "create table user(" +
                "id integer primary key autoincrement," +
                "name varchar(20)," +
                "age integer" +
                ")";
        jdbcTemplate.update(createUser);
        // 2、插入数据
        String insertUserData = "insert into user(name,age) values ('张三',18),('李四',20)";
        jdbcTemplate.update(insertUserData);
        // 3、查询语句
        String selectUserData = "select * from user";
        List<Map<String, Object>> list = jdbcTemplate.queryForList(selectUserData);
        for (Map<String, Object> map : list) {
            for (Map.Entry<String, Object> entry : map.entrySet()) {
                System.out.println(entry.getKey() + "=" + entry.getValue());
            }
        }
        // 5、删除整张表
        String dropTable = "drop table user";
        jdbcTemplate.update(dropTable);
    
```

