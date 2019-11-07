---
title: springboot整合JTA丶atomikos实现XA分布式事务事务方式二推荐
date: 2019-09-04 08:14:08
tags: [SpringBoot,分布式事务]
---

# springboot整合JTA丶atomikos实现XA分布式事务事务方式二 推荐

# pom文件添加 

`跟方式一的一样就不写了`

<!--more-->

# 创建配置文件

```java
server:
  port: 8400

spring:
  application:
    name:  simple-atomikos-demo
  jta:
    enabled: true
    atomikos:
      connectionfactory:
        borrow-connection-timeout: 2000
      datasource:
        test1:
          unique-resource-name: dataSourceOne
          max-pool-size: 5 # The maximum size of the pool.
          min-pool-size: 1  # The minimum size of the pool.
          max-life-time: 20000 # The time, in seconds, that a connection can be pooled for before being destroyed. 0 denotes no limit.
          max-idle-time: 60 # The time, in seconds, after which connections are cleaned up from the pool.
          maintenance-interval: 60 # The time, in seconds, between runs of the pool's maintenance thread.
          borrow-connection-timeout: 10000  # Timeout, in seconds, for borrowing connections from the pool.
          reap-timeout: 0 # The reap timeout, in seconds, for borrowed connections. 0 denotes no limit.
          test-query: SELECT 1
          xa-data-source-class-name: com.mysql.cj.jdbc.MysqlXADataSource
          xa-properties:
            url: jdbc:mysql://127.0.0.1:3339/db_user?Unicode=true&characterEncoding=UTF-8&useSSL=false
            user: root
            password: 123456
            pinGlobalTxToPhysicalConnection : true
        test2:
          unique-resource-name: dataSourceTwo
          max-pool-size: 5 # The maximum size of the pool.
          min-pool-size: 1  # The minimum size of the pool.
          max-life-time: 20000 # The time, in seconds, that a connection can be pooled for before being destroyed. 0 denotes no limit.
          max-idle-time: 60 # The time, in seconds, after which connections are cleaned up from the pool.
          maintenance-interval: 60 # The time, in seconds, between runs of the pool's maintenance thread.
          borrow-connection-timeout: 10000  # Timeout, in seconds, for borrowing connections from the pool.
          reap-timeout: 0 # The reap timeout, in seconds, for borrowed connections. 0 denotes no limit.
          test-query: SELECT 1
          xa-data-source-class-name: com.mysql.cj.jdbc.MysqlXADataSource
          xa-properties:
            url: jdbc:mysql://127.0.0.1:3339/db_account?Unicode=true&characterEncoding=UTF-8&useSSL=false
            user: root
            password: 123456
            pinGlobalTxToPhysicalConnection : true

mybatis:
  type-aliases-package: com.linjingc.jtaatomikosdemo.entity
  # 配置自动转换驼峰标识
  configuration:
    map-underscore-to-camel-case: true

```

需要注意的是 这边还需要要添加一个 jta.properties的文件

```java
om.atomikos.icatch.service=com.atomikos.icatch.standalone.UserTransactionServiceFactory
# https://www.atomikos.com/Documentation/KnownProblems#MySQL_XA_bug
# raised -5: invalid arguments were given for the XA operation
com.atomikos.icatch.serial_jta_transactions=false

```

这里好像是个解决BUG的方法



# 创建数据源配置文件

## dbconfig1

```java
package com.linjingc.simpleatomikosdemo.dbconfig;

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

## dbconfig2

```java
package com.linjingc.simpleatomikosdemo.dbconfig;

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

# 配置mybaitis数据源

## OneDatabaseConfig数据源

```java
package com.linjingc.simpleatomikosdemo.mybaitisconfig;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jta.atomikos.AtomikosDataSourceBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@MapperScan(basePackages = "com.linjingc.simpleatomikosdemo.dao1", sqlSessionFactoryRef = "oneSqlSessionFactory")
public class OneDatabaseConfig {

  @Bean(name = "oneDataSource")
  @ConfigurationProperties(prefix = "spring.jta.atomikos.datasource.test1")
  public DataSource oneDataSource() {
    return new AtomikosDataSourceBean();
  }

  @Bean(name = "oneSqlSessionFactory")
  public SqlSessionFactory oneSqlSessionFactory(@Qualifier("oneDataSource") DataSource oneDataSource) throws Exception {
    SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
    bean.setDataSource(oneDataSource);
    return bean.getObject();
  }
}
```

## TwoDatabaseConfig数据源

```java
package com.linjingc.simpleatomikosdemo.mybaitisconfig;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jta.atomikos.AtomikosDataSourceBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@MapperScan(basePackages = "com.linjingc.simpleatomikosdemo.dao", sqlSessionFactoryRef = "twoSqlSessionFactory")
public class TwoDatabaseConfig {

  @Bean(name = "twoDataSource")
  @ConfigurationProperties(prefix = "spring.jta.atomikos.datasource.test2")
  public DataSource oneDataSource() {
    return new AtomikosDataSourceBean();
  }

  @Bean(name = "twoSqlSessionFactory")
  public SqlSessionFactory oneSqlSessionFactory(@Qualifier("twoDataSource") DataSource oneDataSource) throws Exception {
    SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
    bean.setDataSource(oneDataSource);
    return bean.getObject();
  }
}
```



这样数据源就创建完毕了

# 还是那句话 mapper分开扫描

## dao1

```
package com.linjingc.simpleatomikosdemo.dao1;

import com.linjingc.simpleatomikosdemo.entity.BasicUser;
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

## dao2

```java
package com.linjingc.simpleatomikosdemo.dao;

import com.linjingc.simpleatomikosdemo.entity.Account;
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

# 测试

```java
package com.linjingc.simpleatomikosdemo.service;

import com.linjingc.simpleatomikosdemo.dao.AccountMapper;
import com.linjingc.simpleatomikosdemo.dao1.UserMapper;
import com.linjingc.simpleatomikosdemo.entity.Account;
import com.linjingc.simpleatomikosdemo.entity.BasicUser;
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
   for (int i = 0; i < 100; i++) {
      new Thread(() -> {
        BasicUser user = new BasicUser();
        user.setName("wangxiaoxiao");
        userMapper.insert(user);

//		int i = 1 / 0;//模拟异常，spring回滚后，db_user库中user表中也不会插入记录
        Account account = new Account();
        account.setUserId(user.getId());
        account.setMoney(1111d);
        accountMapper.insert(account);
      }).run();
    }
//		int i = 1 / 0;

  }
}

```

