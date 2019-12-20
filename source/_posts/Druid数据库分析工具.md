---
title: Druid数据库分析工具
date: 2019-11-20 13:49:15
tags: [java,mysql,数据库,调优]
---

# Druid连接池介绍及使用

##  一、 Druid的简介

​           Druid是Java语言中最好的数据库连接池，在功能、性能、扩展性方面，都超过其他数据库连接池，包括DBCP、C3P0、Proxool、JBoss DataSource。Druid已经在阿里巴巴部署了超过600个应用，经过生产环境大规模部署的严苛考验。

​            Druid连接池为监控而生，内置强大的监控功能，监控特性不影响整体性能。功能强大，能防SQL注入，内置Loging能诊断Hack应用行为

## 二、 哪里下载druid

maven中央仓库: http://central.maven.org/maven2/com/alibaba/druid/

## 三、 Github上Druid常见问题

https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98

## 四、 Druid是一个JDBC组件，包括三个部分

基于Filter－Chain模式的插件体系。

DruidDataSource 高效可管理的数据库连接池。

SQLParse

<!--more-->

## 五、 支持哪些数据库

| 数据库    | 支持状态         |
| --------- | ---------------- |
| mysql     | 支持，大规模使用 |
| oracle    | 支持，大规模使用 |
| sqlserver | 支持             |
| postgres  | 支持             |
| db2       | 支持             |
| h2        | 支持             |
| derby     | 支持             |
| sqlite    | 支持             |
| sybase    | 支持             |

## 六、竞品对比

| 功能类别           | 功能            | Druid        | DBCP | Tomcat-jdbc     | C3P0 |
| ------------------ | --------------- | ------------ | ---- | --------------- | ---- |
| 性能               | PSCache         | 是           | 是   | 是              | 是   |
| LRU                | 是              | 是           | 是   | 是              |      |
| SLB负载均衡支持    | 是              | 否           | 否   | 否              |      |
| 稳定性             | ExceptionSorter | 是           | 否   | 否              | 否   |
| 扩展               | 扩展            | Filter       |      | JdbcIntercepter |      |
| 监控               | 监控方式        | jmx/log/http | jmx  | jmx             | jmx  |
| 支持SQL级监控      | 是              | 否           | 否   | 否              |      |
| Spring/Web关联监控 | 是              | 否           | 否   | 否              |      |
|                    | 诊断支持        | LogFilter    | 否   | 否              | 否   |
| 连接泄露诊断       | logAbandoned    | 否           | 否   | 否              |      |
| 安全               | SQL防注入       | 是           | 无   | 无              | 无   |
| 支持配置加密       | 是              | 否           | 否   | 否              |      |

​                   LRU 是一个性能关键指标，特别Oracle，每个Connection对应数据库端的一个进程，如果数据库连接池遵从LRU，有助于数据库服务器优化，这是重要的指标。在测试中，Druid、DBCP、Proxool是遵守LRU的。BoneCP、C3P0则不是。

​                PSCache是数据库连接池的关键指标。在Oracle中，类似SELECT NAME FROM USER WHERE ID = ?这样的SQL，启用PSCache和不启用PSCache的性能可能是相差一个数量级的。Proxool是不支持PSCache的数据库连接池，如果你使用Oracle、SQL Server、DB2、Sybase这样支持游标的数据库，那你就完全不用考虑Proxool。

​            Oracle 10系列的Driver，如果开启PSCache，会占用大量的内存，必须做特别的处理，启用内部的EnterImplicitCache等方法优化才能够减少内存的占用。这个功能只有DruidDataSource有。如果你使用的是Oracle Jdbc，你应该毫不犹豫采用DruidDataSource。

​                  ExceptionSorter是一个很重要的容错特性，如果一个连接产生了一个不可恢复的错误，必须立刻从连接池中去掉，否则会连续产生大量错误。这个特性，目前只有JBossDataSource和Druid实现。Druid的实现参考自JBossDataSource，经过长期生产反馈补充

##  七、 Druid使用

### Maven引入

```
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid</artifactId>
   <version>1.1.9</version>
</dependency>
```

### Druid结合spring配置

```
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="clone">
        <!-- 基本属性driverClassName、 url、user、password -->
        <property name="driverClassName" value="${web.jdbc.driver}" />
        <property name="url" value="${web.jdbc.url}" />
        <property name="username" value="${web.jdbc.username}" />
        <property name="password" value="${web.jdbc.password}" />

        <!-- 配置初始化大小、最小、最大 -->
        <!-- 通常来说，只需要修改initialSize、minIdle、maxActive -->
        <!-- 初始化时建立物理连接的个数，缺省值为0 -->
        <property name="initialSize" value="${web.jdbc.initialSize}" />
        <!-- 最小连接池数量 -->
        <property name="minIdle" value="${web.jdbc.minIdle}" />
        <!-- 最大连接池数量，缺省值为8 -->
        <property name="maxActive" value="${web.jdbc.maxActive}" />

        <!-- 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁  -->
        <property name="maxWait" value="${jdbc.maxWait}" />
        <!--
            有些数据库连接的时候有超时限制（MySQL连接在8小时后断开），或者由于网络中断等原因，连接池的连接会出现失效的情况，这时候可以设置一个testWhileIdle参数为true，
            如果检测到当前连接不活跃的时间超过了timeBetweenEvictionRunsMillis，则去手动检测一下当前连接的有效性，在保证确实有效后才加以使用。
            在检测活跃性时，如果当前的活跃时间大于minEvictableIdleTimeMillis，则认为需要关闭当前连接。当
            然，为了保证绝对的可用性，你也可以使用testOnBorrow为true（即在每次获取Connection对象时都检测其可用性），不过这样会影响性能。
        -->
        <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒（3600000:为1小时） -->
        <property name="timeBetweenEvictionRunsMillis" value="90000" />
        <!-- 配置一个连接在池中最小生存的时间，单位是毫秒(300000:为5分钟) -->
        <property name="minEvictableIdleTimeMillis" value="1800000" />
        <!-- 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn、testWhileIdle都不会其作用。 -->
        <property name="validationQuery" value="${jdbc.validationQuery}" />
        <!-- 申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。建议配置为true，不影响性能，并且保证安全性-->
        <property name="testWhileIdle" value="true" />
        <!-- 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。缺省值:true -->
        <property name="testOnBorrow" value="false" />
        <!-- 归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。缺省值:false -->
        <property name="testOnReturn" value="false" />

        <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
        <!-- 是否缓存preparedStatement，也就是PSCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql5.5以下的版本中没有PSCache功能，建议关闭掉。5.5及以上版本有PSCache，建议开启。缺省值:false -->
        <property name="poolPreparedStatements" value="true" />
        <!-- 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 -->
        <property name="maxPoolPreparedStatementPerConnectionSize" value="50" />

        <!-- 解密密码必须要配置的项 -->
        <property name="filters" value="config" />
        <property name="connectionProperties" value="config.decrypt=true;config.decrypt.key=${web.jdbc.publickey}" />
    </bean>
```

### 数据库密码加密

进入jar包所在目录

java -cp  druid-1.1.9.jar com.alibaba.druid.filter.config.ConfigTools XXXXX

![image-20191220135536967](/img/2019-12-19/druid1.png)

```
web.jdbc.url=jdbc:mysql://XXXX:3306/XXX?useOldAliasMetadataBehavior=true&autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
web.jdbc.username=用户名
web.jdbc.password=password
web.jdbc.publickey=publicKey

若需要使用原始密码
注释connectionProperties 属性即可
```

### 慢sql记录提醒

```
<property name="filters" value="stat,wall" />

<!-- 慢SQL记录 -->
    <bean id="stat-filter" class="com.alibaba.druid.filter.stat.StatFilter">
        <!-- 慢sql时间设置,即执行时间大于200毫秒的都是慢sql -->
        <property name="slowSqlMillis" value="200"/>
        <property name="logSlowSql" value="true"/>
    </bean>

    <bean id="log-filter" class="com.alibaba.druid.filter.logging.Log4jFilter">
        <property name="dataSourceLogEnabled" value="true" />
        <property name="statementExecutableSqlLogEnable" value="true" />
</bean>

```

### 开启web监控

```
<filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    <init-param>
        <param-name>exclusions</param-name>
        <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
    </init-param>
    <init-param>
	    <param-name>profileEnable</param-name>
	    <param-value>true</param-value>
	</init-param>
  </filter>
  <filter-mapping>
    <filter-name>DruidWebStatFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
	<servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
  </servlet>
  <servlet-mapping>
      <servlet-name>DruidStatView</servlet-name>
      <url-pattern>/druid/*</url-pattern>
  </servlet-mapping>
```

### 开启spring类监控

```
<bean id="druid-stat-interceptor" class="com.alibaba.druid.support.spring.stat.DruidStatInterceptor" />
    <bean id="druid-stat-pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut" scope="prototype">
	    <property name="patterns">
			<list>
				<value>com.jf.uhrunit.service.*</value>
				<value>com.jf.uhrunit.queryhelper.*</value>
			</list>
		</property>
	</bean>
	<aop:config proxy-target-class="true">
	<aop:advisor advice-ref="druid-stat-interceptor" pointcut-ref="druid-stat-pointcut" />
	</aop:config>
```



web.jdbc.username=用户名

web.jdbc.password=password

web.jdbc.publickey=publicKey

 若需要使用原始密码
注释connectionProperties 属性即可