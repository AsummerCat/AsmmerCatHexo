---
title: sharding-jdbc的基础使用-分库分表
date: 2019-04-20 18:32:58
tags: [数据库,sharding-jdbc]
---

# sharding-jdbc 数据库中间件

* [官方网站](https://shardingsphere.apache.org/index_zh.html)

* [官方文档](https://shardingsphere.apache.org/document/legacy/3.x/document/cn/manual/sharding-jdbc/configuration/config-java/)
* [demo地址](https://github.com/AsummerCat/shardingjdbcdemo)

## 传统数据库

单表数据库达到1500w 性能开始急剧下降

# 配置方式

两种配置方式

* 利用@Configuration 写入 
* 利用yml配置 写入 

<!--more-->

# Demo 构建

## 注意事项

* 多个数据源的时候 如果需要有不分片的数据表的话 必选设置默认数据源 并且 每个库的表要一样

## 引入pom.xml



```
<!--连接池-->
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.16</version>
    </dependency>


    <!-- sharding-jdbc-core -->
    <dependency>
      <groupId>io.shardingsphere</groupId>
      <artifactId>sharding-jdbc-core</artifactId>
      <version>3.1.0</version>
    </dependency>

```

## jpa的配置文件信息

```
spring:
  jpa:
    database: mysql
    show-sql: true
    hibernate:
      ddl-auto: none
```

## 创建sharding-jdbc数据源的信息

* 逻辑

*  创建配置真实数据源->追加到dataSourceMap中

* 创建sharding-jdbc的分片规则 ->添加分片的节点信息  ->(如果多数据源 配置默认数据源)
* 生成DataSource

```
package com.linjingc.shardingjdbcdemo.config;

import com.alibaba.druid.pool.DruidDataSource;
import io.shardingsphere.api.config.rule.ShardingRuleConfiguration;
import io.shardingsphere.api.config.rule.TableRuleConfiguration;
import io.shardingsphere.api.config.strategy.InlineShardingStrategyConfiguration;
import io.shardingsphere.core.keygen.DefaultKeyGenerator;
import io.shardingsphere.core.keygen.KeyGenerator;
import io.shardingsphere.shardingjdbc.api.ShardingDataSourceFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.AutoConfigureAfter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

import javax.sql.DataSource;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ConcurrentHashMap;

/**
 * sharding-jdbc数据源配置
 *
 * @author cxc
 */
@Configuration
@AutoConfigureAfter(DataSource.class)
public class DataSourceConfiguration {

    @Autowired
    private Environment env;

    @Bean
    public DataSource getDataSource() throws SQLException {
        return buildDataSource();
    }

    private DataSource buildDataSource() throws SQLException {
        // 配置真实数据源
        Map<String, DataSource> dataSourceMap = new HashMap<>(4);
        // 添加两个数据库db0,db1到map里
        dataSourceMap.put("db0", createDataSource("db0"));
        dataSourceMap.put("db1", createDataSource("db1"));


        // 创建user表的分片规则
        TableRuleConfiguration userNodes = createDataNodes();

        // 配置分片规则
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        //添加分片规则
        shardingRuleConfig.getTableRuleConfigs().add(userNodes);


        //配置默认数据源 不分片的数据存入这里
        // 设置默认db为db0，也就是为那些没有配置分库分表策略的指定的默认库
        // 如果只有一个库，也就是不需要分库的话，map里只放一个映射就行了，只有一个库时不需要指定默认库，但2个及以上时必须指定默认库，否则那些没有配置策略的表将无法操作数据
        // *** 这里需要注意的是 两个库的表需要一样 不然排查的时候会找不到表 就无法使用默认数据源**
        shardingRuleConfig.setDefaultDataSourceName("db0");


        // 获取数据源对象
        DataSource dataSource = ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingRuleConfig, new ConcurrentHashMap(), new Properties());

        return dataSource;
    }


    /**
     * 创建数据源
     *
     * @param dataSourceName
     * @return
     */
    private static DataSource createDataSource(final String dataSourceName) {
        // 使用druid连接数据库
        DruidDataSource result = new DruidDataSource();
        result.setDriverClassName("com.mysql.cj.jdbc.Driver");

        result.setUrl(String.format("jdbc:mysql://112.74.43.136:3306/%s", dataSourceName, "?useUnicode=true&amp;characterEncoding=utf8"));
        result.setUsername("root");
        result.setPassword("");
        return result;
    }

    /**
     * 配置分表逻辑的节点
     *
     * @return
     */
    private static TableRuleConfiguration createDataNodes() {
        // 配置user表规则
        TableRuleConfiguration orderTableRuleConfig = new TableRuleConfiguration();
        orderTableRuleConfig.setLogicTable("t_user");
        orderTableRuleConfig.setActualDataNodes("db${0..1}.t_user${0..1}");

        // 配置分库策略
        orderTableRuleConfig.setDatabaseShardingStrategyConfig(new InlineShardingStrategyConfiguration("id", "db${id % 2}"));

        //分表策略
        orderTableRuleConfig.setTableShardingStrategyConfig(new InlineShardingStrategyConfiguration("id", "t_user${id % 2}"));

        return orderTableRuleConfig;
    }


    @Bean
    public KeyGenerator keyGenerator() {
        return new DefaultKeyGenerator();
    }


    //private Properties getProperties() {
    //    Properties p = new Properties();
    //    p.put("minimum-idle", env.getProperty("hikari.minimum-idle"));
    //    p.put("maximum-pool-size", env.getProperty("hikari.maximum-pool-size"));
    //    p.put("auto-commit", env.getProperty("hikari.auto-commit"));
    //    p.put("idle-timeout", env.getProperty("hikari.idle-timeout"));
    //    p.put("max-lifetime", env.getProperty("hikari.max-lifetime"));
    //    p.put("connection-timeout", env.getProperty("hikari.connection-timeout"));
    //    p.put("connection-test-query", env.getProperty("hikari.connection-test-query"));
    //    return p;
    //}
}

```



# sharding-jdbc计算总结

![sharding-jdbc计算总结.png](/img/2019-4-20/sharding-jdbc计算总结.png)

# sharding-jdbc应用层架构图

sharding-jdbc应用层架构图

![sharding-jdbc应用层架构图](/img/2019-4-20/sharding-jdbc应用层架构图.png)