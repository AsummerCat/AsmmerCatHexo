---
title: LiquiBase数据库导入语句工具
date: 2021-01-11 17:19:50
tags: [java,数据库]
---

# LiquiBase数据库导入语句工具
目的是用来 项目中维护增删改数据库结构和语句的一个管理工具  
方便后期维护,回滚之类的

Liquibase是一个用于跟踪、管理和应用数据库变化的开源的数据库重构工具。它将所有数据库的变化（包括结构和数据）都保存在XML文件中，便于版本控制

## 运行ChangeSet  跑批

　　有很多方式都可以运行change log 包括：  
　　[command line](https://docsstage.liquibase.com/tools-integrations/cli/home.html),   
　　[Ant](https://docs.liquibase.com/tools-integrations/ant/home.html),  
　　[Maven](https://docs.liquibase.com/tools-integrations/maven/home.html),   
　　[Spring](https://docs.liquibase.com/tools-integrations/springboot/using-springboot-with-maven.html)

<!--more-->　
　
执行命令

```
 mvn liquibase:update
```

## 注意
会在数据库记录两张表 用来控制回滚和日志的
```
DATABASECHANGELOG
DATABASECHANGELOGLOCK
```

# maven模式
## [Maven模式demo](https://github.com/AsummerCat/LiquiBaseDemo)

### 第一步 导入pom
```
   <!--liquibase核心包 -->
        <dependency>
            <groupId>org.liquibase</groupId>
            <artifactId>liquibase-core</artifactId>
            <version>4.2.2</version>
        </dependency>
        <!--liquibase maven构建-->
        <dependency>
            <groupId>org.liquibase</groupId>
            <artifactId>liquibase-maven-plugin</artifactId>
            <version>4.2.2</version>
        </dependency>
        <!--数据库连接-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>

```
接着 修改maven的构建模式
```
   <!--这里用来执行  mvn liquibase:update 语法-->
    <build>
        <pluginManagement>
            <plugins>
                    <plugin>
                        <groupId>org.liquibase</groupId>
                        <artifactId>liquibase-maven-plugin</artifactId>
                        <configuration>
                            <verbose>true</verbose>
                            <promptOnNonLocalDatabase>false</promptOnNonLocalDatabase>
                            <!--配置文件地址-->
                            <propertyFile>${project.basedir}/src/main/resources/liquibase.properties</propertyFile>
                        </configuration>
                    </plugin>
            </plugins>
        </pluginManagement>
    </build>
```
### 第二步创建文件
在`resource`下创建两个文件
`liquibase.properties`和`db-changelog-master.xml`

#### liquibase.properties
这里是引入change-log的主文件 和数据库地址
```
changeLogFile=db-changelog-master.xml
driver: com.mysql.jdbc.Driver
url: jdbc:mysql://localhost:3339/db_user?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false
username:root
password:123456

```
#### db-changelog-master.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.2.xsd">
    <!--根据path读取resource下的change-set-->
    <includeAll path="data/202001" relativeToChangelogFile="true" errorIfMissingOrEmpty="true"/>
    <includeAll path="data/202002" relativeToChangelogFile="true" errorIfMissingOrEmpty="true"/>
    <includeAll path="data/202003" relativeToChangelogFile="true" errorIfMissingOrEmpty="true"/>

</databaseChangeLog>

```
在每个`includeAll`的`path`对应的文件夹中需要创建具体的语法xml
例如:
`db_user.xml`

```
<?xml version="1.0" encoding="UTF-8"?>

<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.2.xsd">

    <changeSet id="3" author="bob">
        <sql>
            <!--直接插入语句-->
            delete from user where id =110;
        </sql>
    </changeSet>

</databaseChangeLog>

```

这样就构建出一个LinquiBase的数据库导入了
最后执行
`mvn liquibase:update` 这样就开始跑语句了
注意需要先`mvn compile`

# 控制台模式 
在服务器上没有maven或者其他构建工具的话
可以直接使用命令行操作
```

java -jar liquibase.jar \
      --driver=oracle.jdbc.OracleDriver \
      --classpath=\path\to\classes:jdbcdriver.jar \
      --changeLogFile=com/example/db.changelog.xml \
      --url="jdbc:oracle:thin:@localhost:1521:oracle" \
      --username=scott \
      --password=tiger \
      update
```
如果是用默认配置,直接执行以下语句就可以了
```
java -jar liquibase.jar update
```