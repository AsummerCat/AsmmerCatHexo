---
title: Myabtis源码解析-集成spring都做了什么三
date: 2019-09-26 15:37:52
tags: [mybatis源码解析]
---

# Myabtis源码解析-集成spring都做了什么

## [demo地址](https://github.com/AsummerCat/mybatis-demo/tree/master/mybaitis-source-code-demo)

以[mybatis-spring-2.0.2 ](https://github.com/mybatis/spring/tree/mybatis-spring-2.0.2)为例，工程划分六个模块。

## 1、annotation 模块

![annotation模块](/img/2019-09-15/2.png)

　定义了@MapperScan和@MapperScans，用于扫描mapper接口。以及mapper扫描注册器（MapperScannerRegistrar），扫描注册器实现了 ImportBeanDefinitionRegistrar接口， 在Spring容器启动时会运行所有实现了这个接口的实现类，
注册器内部会注册一系列MyBatis相关Bean。

## 2、batch 模块

![batch 模块](/img/2019-09-15/3.png)

　　批处理相关，基于优秀的批处理框架Spring batch 封装了三个批处理相关类：

- MyBatisBatchItemWriter（批量写）
- MyBatisCursorItemReader（游标读）
- MyBatisPagingItemReader（分页读）

　　在使用Mybatis时，方便的应用Spring  batch

<!--more-->

## 3、config模块

![config模块](/img/2019-09-15/4.png)

解析、处理读取到的配置信息。



## 4、mapper模块

![mapper模块](/img/2019-09-15/5.png)

这里是处理mapper的地方：ClassPathMapperScanner（根据路径扫描Mapper接口）与MapperScannerConfigurer 配合，完成批量扫描mapper接口并注册为MapperFactoryBean。

## 5、support 模块

![support 模块](/img/2019-09-15/6.png)

　　支持包，`SqlSessionDaoSupport` 是一个抽象的支持类，用来为你提供 SqlSession调用getSqlSession()方法会得到一个SqlSessionTemplate。

## 6.transaction 模块

![transaction 模块](/img/2019-09-15/7.png)

# 初始化相关

## SqlSessionFactoryBean

在基础的MyBatis中，通过`SqlSessionFactoryBuilder`创建`SqlSessionFactory。`集成Spring后由SqlSessionFactoryBean来创建。　

```
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>...
```

需要注意`SqlSessionFactoryBean`实现了Spring的`FactoryBean`接口。这意味着由Spring最终创建不是`SqlSessionFactoryBean`本身，而是 getObject()的结果。我们来看下getObject()

```
@Override
  public SqlSessionFactory getObject() throws Exception {
    if (this.sqlSessionFactory == null) {
      //配置加载完毕后，创建SqlSessionFactory
      afterPropertiesSet();
    }
    return this.sqlSessionFactory;
  }
```

getObject()最终返回了当前类的 SqlSessionFactory，因此，Spring 会在应用启动时创建 `SqlSessionFactory`，并以 `sqlSessionFactory名称`放进容器。

## 两个重要属性

1. **SqlSessionFactory** 有一个唯一的必要属性：用于 JDBC 的 `DataSource不能为空，这点在afterPropertisSet（）中体现。`
2. **configLocation**，它用来指定 MyBatis 的 XML 配置文件路径。通常只用来配置 `<settings>相关。其他均使用Spring方式配置`

```
 ```
public void afterPropertiesSet() throws Exception {
    //dataSource不能为空
    notNull(dataSource, "Property 'dataSource' is required");
    //有默认值，初始化 = new SqlSessionFactoryBuilder()
    notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
    //判断configuration && configLocation有且仅有一个
    state((configuration == null && configLocation == null) || 
　　　　　　　　　　!(configuration != null && configLocation != null),
        "Property 'configuration' and 'configLocation' can not specified with together");
    //调用build方法创建sqlSessionFactory
    this.sqlSessionFactory = buildSqlSessionFactory();
  }
```
  

　　　 buildSqlSessionFactory()方法比较长所以，这里省略了一部分代码，只展示主要过程，看得出在这里进行了Mybatis相关配置的解析，完成了Mybatis核心配置类Configuration的创建和填充，最终返回SqlSessionFactory。

```
protected SqlSessionFactory buildSqlSessionFactory() throws Exception {

    final Configuration targetConfiguration;
    
    XMLConfigBuilder xmlConfigBuilder = null;
　 　// 如果自定义了 Configuration，就用自定义的
    if (this.configuration != null) {
      targetConfiguration = this.configuration;
      if (targetConfiguration.getVariables() == null) {
        targetConfiguration.setVariables(this.configurationProperties);
      } else if (this.configurationProperties != null) {
        targetConfiguration.getVariables().putAll(this.configurationProperties);
      }
　　  // 如果配置了原生配置文件路径，则根据路径创建Configuration对象
    } else if (this.configLocation != null) {
      xmlConfigBuilder = new XMLConfigBuilder(this.configLocation.getInputStream()
　　　　　　　　, null, this.configurationProperties);
      targetConfiguration = xmlConfigBuilder.getConfiguration();
    } else {21 　　　// 兜底，使用默认的
      targetConfiguration = new Configuration();
　　　//如果configurationProperties存在，设置属性 
　　  Optional.ofNullable(this.configurationProperties).ifPresent(targetConfiguration::setVariables); }
//解析别名，指定包　　　
if (hasLength(this.typeAliasesPackage)) {
  scanClasses(this.typeAliasesPackage, this.typeAliasesSuperType).stream()
      .filter(clazz -> !clazz.isAnonymousClass())
　　　　　 .filter(clazz -> !clazz.isInterface())
      .filter(clazz -> !clazz.isMemberClass())
　　　　　 .forEach(targetConfiguration.getTypeAliasRegistry()::registerAlias);
}
//解析插件
if (!isEmpty(this.plugins)) {
  Stream.of(this.plugins).forEach(plugin -> {
    targetConfiguration.addInterceptor(plugin);38 }
    ...
//如果需要解决原生配置文件，此时开始解析（即配置了configLocation）
if (xmlConfigBuilder != null) {
  try {
    xmlConfigBuilder.parse();
　　 ...  //有可能配置多个，所以遍历处理（2.0.0支持可重复注解）
if (this.mapperLocations != null) {
    if (this.mapperLocations.length == 0) {
　　　　　　for (Resource mapperLocation : this.mapperLocations) {
       ...  //根据mapper路径，加载所以mapper接口
          XMLMapperBuilder xmlMapperBuilder = new XMLMapperBuilder(mapperLocation.getInputStream(),
              targetConfiguration, mapperLocation.toString(), targetConfiguration.getSqlFragments());
          xmlMapperBuilder.parse(); 
      　//构造SqlSessionFactory
  return this.sqlSessionFactoryBuilder.build(targetConfiguration);
}
```

# **二、事务管理**

MyBatis-Spring 允许 MyBatis 参与到 Spring 的事务管理中。 借助 Spring 的 DataSourceTransactionManager 实现事务管理。　　

​```
/** 一、XML方式配置 **/
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <constructor-arg ref="dataSource" />
</bean>
/** 一、注解方式配置 **/
@Bean
public DataSourceTransactionManager transactionManager() {
  return new DataSourceTransactionManager(dataSource());
}
注意：为事务管理器指定的 DataSource 必须和用来创建 SqlSessionFactoryBean 的是同一个数据源，否则事务管理器就无法工作了。
```

配置好 Spring 的事务管理器，你就可以在 Spring 中按你平时的方式来配置事务。并且支持 @Transactional 注解（**声明式事务**）和 AOP 风格的配置。在事务处理期间，一个单独的 `SqlSession` 对象将会被创建和使用。当事务完成时，这个 session 会以合适的方式提交或回滚。无需DAO类中无需任何额外操作，MyBatis-Spring 将透明地管理事务。

## 2) 编程式事务：

　　推荐`TransactionTemplate` 方式，简洁，优雅。可省略对 `commit` 和 `rollback` 方法的调用。　　　　

```
TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
transactionTemplate.execute(txStatus -> {
  userMapper.insertUser(user);
  return null;
});
  注意：这段代码使用了一个映射器，换成SqlSession同理。
```

# 三、SqlSession

　　在MyBatis 中，使用 `SqlSessionFactory` 来创建 `SqlSession`。通过它执行映射的sql语句，提交或回滚连接，当不再需要它的时候，可以关闭 session。使用 MyBatis-Spring 之后，我们不再需要直接使用 `SqlSessionFactory` 了，因为我们的bean 可以被注入一个线程安全的 `SqlSession`，它能基于 Spring 的事务配置来自动提交、回滚、关闭 session。

## SqlSessionTemplate 　

　　SqlSessionTemplate 是SqlSession的实现，是线程安全的，因此可以被多个DAO或映射器共享使用。也是 MyBatis-Spring 的核心。

# 四、映射器

## **1) 映射器的注册**　　

```
/** 
 *@MapperScan注解方式 
 */
@Configuration
@MapperScan("org.mybatis.spring.sample.mapper")
public class AppConfig {
}
/** 
 *@MapperScanS注解 (since 2.0.0新增，java8 支持可重复注解)
 * 指定多个路径可选用此种方式
 */
@Configuration
@MapperScans({@MapperScan("com.zto.test1"), @MapperScan("com.zto.test2.mapper")})
public class AppConfig {
}
```

```
<!-- MapperScannerConfigurer方式，批量扫描注册 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="com.zto.test.*" />
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
</bean>
```

　无论使用以上哪种方式注册映射器，最终mapper接口都将被注册为MapperFactoryBean。既然是FactoryBean,我们来跟它的getObject()方法看下。

## 2) MapperFactoryBean源码解析

### 1.查找MapperFactoryBean.getObject()　　

```
/**
   * 通过接口类型，获取mapper
   * {@inheritDoc}
   */
  @Override
  public T getObject() throws Exception {
    //getMapper 是一个抽象方法
    return getSqlSession().getMapper(this.mapperInterface);
  }

```

### 2.查看实现类，SqlSessionTemplate.getMapper()

　　　　( 为什么是SqlSessionTemplate，而不是默认的DefaultSqlSession？SqlSessionTemplate是整合包的核心，是线程安全的SqlSession实现，是我们@Autowired mapper接口编程的基础 )

```
 @Override
 public <T> T getMapper(Class<T> type) {
 return getConfiguration().getMapper(type, this);
  }

```

### 3.调用Configuration.getMapper()　　

```
 public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  return mapperRegistry.getMapper(type, sqlSession);
}

```

### 4.调用MapperRegistry.getMapper()　　　

```
@SuppressWarnings("unchecked")
  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
      return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
      throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
}

```

### 5.调用MapperProxyFactory.newInstance()　　

```
@SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

```

　最终看到动态代理生成了一个新的代理实例返回了，也就是说，我们使用@Autowired 注解进来一个mapper接口，每次使用时都会由代理生成一个新的实例。

　　　　**为什么在Mybatis中SqlSession是方法级的，Mapper是方法级的，在集成Spring后却可以注入到类中使用？**

　　　　因为在Mybatis-Spring中所有mapper被注册为FactoryBean，每次调用都会执行getObject()，返回新实例。

# 五、总结

　　　　MyBatis集成Spring后，Spring侵入了Mybatis的初始化和mapper绑定，具体就是：

　　　　1）Cofiguration的实例化是读取Spring的配置文件（注解、配置文件），而不是mybatis-config.xml

　　　　2）mapper对象是方法级别的，Spring通过FactoryBean巧妙地解决了这个问题

　　　　3）事务交由Spring管理