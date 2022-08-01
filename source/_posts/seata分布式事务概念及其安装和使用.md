---
title: seata分布式事务概念及其安装和使用
date: 2020-05-22 14:19:31
tags: [分布式事务,seata]
---

# seata分布式事务概念及其安装和使用

==当前版本基于0.9.0==

```
需要注意的是1.0.0以上配置有可能会有点不一样
自动导入nacos的数据 ->在0.9.0的脚本里
```

## demo

[demo地址](https://github.com/AsummerCat/seata-demo)

## 核心概念

Transaction ID (XID)
```
全局唯一的事务ID
```

RM(ResourceManager 资源管理者)
```
理解为 我们的一个个的微服务 也叫做 事务的参与者
```
TM(TranctionManager 事务管理者)
```
也是我们的一个微服务 ->
充当全局事务的发起者(决定了全局会务的开启,回滚,提交)

*** 方式我们的微服务中标注了@GlobalTransactional,
那么该微服务就会被看成一个TM.
我们业务场景中订单微服务就是一个事务发起者,同时也是一个RM
```

TC(全局事务的协调者)
```
这里就是我们的Seata-server 用来保存全局事务,分钟事务,全局锁等高几率,然后会通知各个RM进行回滚或者提交
也就是要创建一个服务TC 来协调各个服务
```
<!--more-->

## 处理过程
1. TM向TC申请开启一个全局事务,全局事务创建成功并生成一个全局唯一的XID;
2. XID在微服务调用链路的上下文中传播
3. RM向TC注册分支事务,将其纳入XID对应全局事务的管辖;
4. TM向TC发起针对XID的全局提交或者回滚决议
5. TC调度XID下管辖的全局分支事务完成提交或回滚请求

## seata安装及其发布地址
[发布地址](https://github.com/seata/seata/releases)
[官网下载地址](seata.io/zh-cn/blog/download.html)

#### 第一步 解压zip
==解压到指定目录==

#### 第二步 创建库seata 和表
==在seata库里建表==
 库名: seata
 建表语句: ==conf目录中的sb_store.sql(没有的话官网的找)==

#### 第三步 修改conf目录下的file.conf配置文件

==主要修改:自定义事务组名称+事务日志存储模式为db+数据库连接信息==

service模块(服务模块):
修改内容:
```
service {
     vgroup_mapping.(这个位置对应你服务的分组)="这里是注册中心的分组"
    vgroup_mapping.my_test_tx_group="default"
}
```
store模块(存储模块):
修改内容:
```
store{
    ## 存储方式为db
     mode= "db"
    db{
        数据库连接信息相关信息
    } 
}

```
#### 第四步 修改conf目录下的registry.conf配置文件
```
registry{
    ## 选择注册到哪个地方 (注册中心)
    type="nacos"
  ## 并且配置相关信息
    nacos{
        serverAddr="localhost"
        namespace=""
        cluster="default"
    }
}

```
#### 第五步 启动步骤
启动项目 (注册中心)

再启动seata-server ==脚本位置: bin/seata-server.bat==

#### 第六步 创建回滚日志表

在各个库中创建对应的回滚日志表
==conf目录下的:db_undo_log.sql==


## 项目中如何使用

#### 第一步 导入seata的pom
==注意:需要跟我们的seata-server匹配版本==
```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

#### 第二步 修改yml配置 添加seata事务组
```
spring: 
  cloud:
    alibaba:
      seata:
        #自定义事务组名称 需要与seata-server中的对应
        tx-service-group: my_test_tx_group

```

#### 第三步 在resources目录下创建file.conf
客户端配置
```
跟seata-server 差不多内容 直接复制一份
需要修改的部分内容:
service {
    
    vgroup_mapping.自定义分组名称(这里是seata-server写的那个分组)="default"
}
```
#### 第四步 在resources目录下创建registry.conf
客户端配置
```
跟seata-server 差不多内容 直接复制一份
表示注册到哪个位置
```

#### 第五步 注意需要配置数据源

```
package com.linjingc.paydemo.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.transaction.SpringManagedTransactionFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

import javax.sql.DataSource;

/**
 * seata数据源配置
 */
@Configuration
public class DataSourceConfig {
	@Bean
	@ConfigurationProperties(prefix = "spring.datasource")
	public DataSource druidDataSource() {
		return new DruidDataSource();
	}

	@Bean
	public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
		SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
		factoryBean.setDataSource(dataSource);
		factoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
		factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath*:/mapper/*xml"));
		return factoryBean.getObject();
	}
}
```



#### 第六步 业务使用

```
入口方法 添加->
在业务逻辑上添加@GlobalTransactional注解

@GlobalTransactional(
timeoutMills=超时时间(有默认的),
name="该事务的全局名称",
rollbackFor={xxx.class 哪些异常需要回滚},
norollbackFor={xxx.class,哪些异常不需要回滚}

)
```

#### 第七步 在查询库存的方法上加入@GlobalLock 读隔离 防止脏读

```

@GlobalLock
public String select(String productId){

}
这样会在全局锁中->(lock_table)加入一个productId=1的记录 下次别人来读取就堵塞
```

## 原理 

```
在保存的时候 ->会有一个快照 
如果失败了 就将快照恢复给数据库
```
开源版本:seata ,商用版本:GTS

默认使用AT模式: 无侵入的自动补偿机制

AT模式:二阶段提交
```
一阶段: 业务数据和回滚日志记录在同一个本地事务中提交,释放本地锁和连接资源

二阶段: 
   提交异步化,非常快速地完成.
   回滚通过一阶段的回滚日志进行方向补偿
```

### 核心:
在一阶段:
```
Seata会拦截"业务SQL"
1.解析SQL语义,找到"业务SQL"要更新的业务数据,在业务数据被更新前,将其保存成"before image",
2.执行"业务SQL"更新业务数据,在业务数据更新之后,
3.其保存成"after image",最后生成行锁
以上操作全部在一个数据库事务内完成,这样保证了一阶段操作的原子性
```

#### 如果二阶段提交成功
```
 Seata框架只需将一阶段保存的快照数据和行锁删掉,完成数据清理即可;
```

#### 如果二阶段回滚
```
如果回滚的话 seata就需要回滚一阶段已经执行的"业务SQL",还原数据
回滚方式使用"before image"还原业务数据;但在还原前要校验脏写,
对比"数据库当前业务数据"和"after image",
如果两份数据完全一致就说明没有脏写,可以还原业务数据,
如果不一致就说明有脏写,出现脏写就需要转人工处理;

```
回滚流程: 

1.检验脏写("after image " vs "数据库数据")

2.还原数据("before image"-> "逆向SQL"->"数据还原")

3.删除中间数据(删除"after image ","before image",行锁)