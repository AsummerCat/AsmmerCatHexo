---
title: springboot整合JTA丶atomikos实现XA分布式事务事务方式一
date: 2019-09-04 08:13:18
tags: [SpringBoot,分布式事务]
---

# springboot整合JTA丶atomikos实现XA分布式事务事务方式一

[demo地址](https://github.com/AsummerCat/xxa-transactions-demo/tree/9d9b8c8c76f61d8524440293a477bda3e00712b2/jta-atomikos-demo)

# 导入pom

这边使用mybaitis+mysql+atomikos

```java
 <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.17</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <!-- 排除spring boot默认使用的tomcat，使用jetty -->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.1</version>
        </dependency>
        <!--atomikos-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jta-atomikos</artifactId>
            <version>2.1.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.0.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

<!--more-->

# 修改配置文件创建两个数据源

```java
server:
  port: 8200

spring:
  application:
    name:  jta-atomikos-demo
    #JTA开启
  jta:
    enabled: true


mybatis:
  type-aliases-package: com.linjingc.jtaatomikosdemo.entity
  # 配置自动转换驼峰标识
  configuration:
    map-underscore-to-camel-case: true

mysql:
    datasource:
      test1:
        url: jdbc:mysql://127.0.0.1:3339/db_user?Unicode=true&characterEncoding=UTF-8&useSSL=false
        username: root
        password: 123456
      test2:
        url: jdbc:mysql://127.0.0.1:3339/db_account?Unicode=true&characterEncoding=UTF-8&useSSL=false
        username: root
        password: 123456


```

# 创建两个配置类

```
package com.linjingc.jtaatomikosdemo.dbconfig;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@Data
@ConfigurationProperties(prefix = "mysql.datasource.test1")
public class DBConfig1 {
  private String url;
  private String username;
  private String password;
}

```

```java
package com.linjingc.jtaatomikosdemo.dbconfig;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Configuration
@Data
@ConfigurationProperties(prefix = "mysql.datasource.test2")
public class DBConfig2 {
  private String url;
  private String username;
  private String password;
}

```

# 同时也创建两个mybaitis数据源的配置类

```java
package com.linjingc.jtaatomikosdemo.mybaitisconfig;

import com.linjingc.jtaatomikosdemo.dbconfig.DBConfig1;
import com.mysql.cj.jdbc.MysqlXADataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.jta.atomikos.AtomikosDataSourceBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;

@Configuration
// basePackages 最好分开配置 如果放在同一个文件夹可能会报错
@MapperScan(basePackages = "com.linjingc.jtaatomikosdemo.dao1", sqlSessionTemplateRef = "testSqlSessionTemplate")
public class MyBatisConfig1 {
 
  // 配置数据源
  @Bean(name = "testDataSource")
  public DataSource testDataSource(DBConfig1 testConfig) throws SQLException {
    MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
    mysqlXaDataSource.setUrl(testConfig.getUrl());
    mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
    mysqlXaDataSource.setPassword(testConfig.getPassword());
    mysqlXaDataSource.setUser(testConfig.getUsername());
    mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
 
    // 将本地事务注册到创 Atomikos全局事务
    AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
    xaDataSource.setXaDataSource(mysqlXaDataSource);
    xaDataSource.setUniqueResourceName("testDataSource");
    return xaDataSource;
  }
 
  @Bean(name = "testSqlSessionFactory")
  public SqlSessionFactory testSqlSessionFactory(@Qualifier("testDataSource") DataSource dataSource)
			throws Exception {
    SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
    bean.setDataSource(dataSource);
    return bean.getObject();
  }
 
  @Bean(name = "testSqlSessionTemplate")
  public SqlSessionTemplate testSqlSessionTemplate(
			@Qualifier("testSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
    return new SqlSessionTemplate(sqlSessionFactory);
  }
}

```

```java
package com.linjingc.jtaatomikosdemo.mybaitisconfig;

import com.linjingc.jtaatomikosdemo.dbconfig.DBConfig2;
import com.mysql.cj.jdbc.MysqlXADataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.jta.atomikos.AtomikosDataSourceBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.sql.SQLException;

@Configuration
@MapperScan(basePackages = "com.linjingc.jtaatomikosdemo.dao", sqlSessionTemplateRef = "test2SqlSessionTemplate")
public class MyBatisConfig2 {

  // 配置数据源
  @Bean(name = "test2DataSource")
  public DataSource testDataSource(DBConfig2 testConfig) throws SQLException {
    MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
    mysqlXaDataSource.setUrl(testConfig.getUrl());
    mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
    mysqlXaDataSource.setPassword(testConfig.getPassword());
    mysqlXaDataSource.setUser(testConfig.getUsername());
    mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
    AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
    xaDataSource.setXaDataSource(mysqlXaDataSource);
    xaDataSource.setUniqueResourceName("test2DataSource");
    return xaDataSource;
  }

  @Bean(name = "test2SqlSessionFactory")
  public SqlSessionFactory testSqlSessionFactory(@Qualifier("test2DataSource") DataSource dataSource)
			throws Exception {
    SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
    bean.setDataSource(dataSource);
    return bean.getObject();
  }

  @Bean(name = "test2SqlSessionTemplate")
  public SqlSessionTemplate testSqlSessionTemplate(
			@Qualifier("test2SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
    return new SqlSessionTemplate(sqlSessionFactory);
  }
}

```

需要注意的是两个 mapper不能放在同一个地方 避免 数据源错误

分别存在mapper 然后处理包扫描

## dao包

```
package com.linjingc.jtaatomikosdemo.dao;

import com.linjingc.jtaatomikosdemo.entity.Account;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Component;

@Mapper
@Component
public interface AccountMapper {
    @Insert("INSERT INTO account(user_id,money) VALUES(#{userId},#{money})")
    public void insert(Account account);
}
```

## dao1包

```java
package com.linjingc.jtaatomikosdemo.dao1;

import com.linjingc.jtaatomikosdemo.entity.BasicUser;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Options;
import org.springframework.stereotype.Component;

/**
 * @author cxc
 */
@Component
@Mapper
public interface UserMapper {
  @Insert("INSERT INTO user(id,name) VALUES(#{id},#{name})")
  @Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
  public void insert(BasicUser user);
  /**
	 * 需要注意的内容
	 * #{name} 和${name}的区别    #{}代表自动拼接``  ${}表示需要手动添加``
	 */
}

```

这样基本都搭建成功了

现在只要声明spring的事务注解就可以生效xa事务了

```java
package com.linjingc.jtaatomikosdemo.service;

import com.linjingc.jtaatomikosdemo.dao.AccountMapper;
import com.linjingc.jtaatomikosdemo.dao1.UserMapper;
import com.linjingc.jtaatomikosdemo.entity.Account;
import com.linjingc.jtaatomikosdemo.entity.BasicUser;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;

@Service
public class JTAService {
  //操作db_account库
  @Resource
  private AccountMapper accountMapper;
  //操作db_user库
  @Resource
  private UserMapper userMapper;


  @Transactional
  public void insert() {
    BasicUser user = new BasicUser();
    user.setName("wangxiaoxiao");
    userMapper.insert(user);

        int i = 1 / 0;//模拟异常，spring回滚后，db_user库中user表中也不会插入记录
    Account account = new Account();
    account.setUserId(user.getId());
    account.setMoney(1111d);
    accountMapper.insert(account);
  }
}

```

