---
title: 分布式事务框架seata概念
date: 2020-03-21 18:49:29
tags: [分布式事务,seata]

---

# 阿里推出的一个分布式事务框架

文档: https://seata.io/zh-cn/docs/overview/terminology.html

## 核心概念

### RM(ResourceManager 资源管理者)

```
理解为 我们的一个个的微服务 也叫做 事务的参与者
```

### TM(TranctionManager 事务管理者)

```
也是我们的一个微服务 -> 充当全局事务的发起者(决定了全局会务的开启,回滚,提交)
*** 方式我们的微服务中标注了@GlobalTransactional,那么该微服务就会被看成一个TM. 我们业务场景中订单微服务就是一个事务发起者,同时也是一个RM
```

### TC(全局事务的协调者)

```
这里就是我们的Seata-server 用来保存全局事务,分钟事务,全局锁等高几率,然后会通知各个RM进行回滚或者提交
```

<!--more-->

也就是要创建一个服务TC 来协调各个服务  

# 启动TC流程

## 去github下载一个TC 也就是seata-Server

https://github.com/seata/seata/releases

## 修改配置 conf文件

### file.conf  文件相关配置

```
修改 service{}里面的内容

## 分组名称
vgroup_mapping.prex_tx_group=" 你的分组名称"

## TC服务地址
default.grouplist="127.0.0.1:8091"
```

```
事务日志存储
修改 stare{}里面的内容

## 默认是存储在文件的 修改为db    模式: file ,db
mode="db"


## 修改db{}里面的内容

这里就是创建db存储相关的部分 比如数据库地址 还有最大最小连接
  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "mysql"
    password = "mysql"
    minConn = 1
    maxConn = 10
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
  }

```

### registry.conf  注册相关配置

```
## 修改registry{}  
    ## 修改注册中心的类型    file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
   type="nacos"
   
   
```

## 然后在db中创建seata相关的表

SEATA AT 模式需要 `UNDO_LOG` 表

每个物理库都需要创建这个表

```
unlog 记录进行回滚的表

-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

```

## 演示实例

https://github.com/seata/seata-samples



# 启动微服务

### 引入相关seata的pom

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

## 添加Tm注解 @GlobalTransactional 开启全局事务

```
@GlobalTransactional(name="create-order",rollbackFor=Exception.class,timeoutMills = 300000)
默认超时时间为60s name为事务方法签名的别名 默认为空 注释内参数均可忽略
```

## 在查询库存的方法上加入@GlobalLock 读隔离 防止脏读

```
@GlobalLock
public String select(String productId){

}
这样会在全局锁中->(lock_table)加入一个productId=1的记录 下次别人来读取就堵塞
```



