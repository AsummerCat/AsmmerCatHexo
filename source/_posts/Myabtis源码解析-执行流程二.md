---
title: Myabtis源码解析-执行流程二
date: 2019-09-26 15:36:56
tags: [mybatis源码解析]
---

# Myabtis源码解析-执行流程

## [demo地址](https://github.com/AsummerCat/mybatis-demo/tree/master/mybaitis-source-code-demo)

[参考地址](https://blog.csdn.net/luanlouis/article/details/40422941)

![流程图](/img/2019-09-15/1.png)

<!--more-->

## 核心组件

```
SqlSession            作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
Executor              MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
StatementHandler   封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。
ParameterHandler   负责对用户传递的参数转换成JDBC Statement 所需要的参数，
ResultSetHandler    负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
TypeHandler          负责java数据类型和jdbc数据类型之间的映射和转换
MappedStatement   MappedStatement维护了一条<select|update|delete|insert>节点的封装， 
SqlSource            负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
BoundSql             表示动态生成的SQL语句以及相应的参数信息
Configuration        MyBatis所有的配置信息都维持在Configuration对象之中。xxxxxxxxxx Mybatis的核心部件SqlSession            作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能Executor              MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护StatementHandler   封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数、将Statement结果集转换成List集合。ParameterHandler   负责对用户传递的参数转换成JDBC Statement 所需要的参数，ResultSetHandler    负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；TypeHandler          负责java数据类型和jdbc数据类型之间的映射和转换MappedStatement   MappedStatement维护了一条<select|update|delete|insert>节点的封装， SqlSource            负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回BoundSql             表示动态生成的SQL语句以及相应的参数信息Configuration        MyBatis所有的配置信息都维持在Configuration对象之中。
```



## 简化流程：

   第一步：
 		获取XML文件流对象（Resources）
   第二步：

```
SqlSessionFactoryBuilder->build
```

   		创建一个带有Configuration对象的DefaultSqlSessionFactory对象
  		此对象用来实例化SqlSession的对象。
   第三步：
 	       由DefaultSqlSessionFactory创建SqlSession带有Configuration对象，执行器对象(Executor(Transaction)

# 流程代码

## **SqlSession的获取**

```
// 1.读取config目录下Configure.xml文件
Reader reader = Resources.getResourceAsReader("config/Configure.xml");
// 2.使用SqlSessionFactoryBuilder创建SqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
// 3.在SqlSessionFactory中创建SqlSession
SqlSession session = sqlSessionFactory.openSession();
```

## **Resources.getResourceAsReader("config/Configure.xml")读取文件**

```
  public static Reader getResourceAsReader(String resource) throws IOException {
    Reader reader;
    if (charset == null) {
      // 默认为null，走该步骤
      reader = new InputStreamReader(getResourceAsStream(resource));
    } else {
      reader = new InputStreamReader(getResourceAsStream(resource), charset);
    }
    return reader;
  }
 
//getResourceAsStream()
  public static InputStream getResourceAsStream(String resource) throws IOException {
    return getResourceAsStream(null, resource);
  }
 
//getResourceAsStream()
  public static InputStream getResourceAsStream(ClassLoader loader, String resource) throws IOException {
    // ClassLoaderWrapper classLoaderWrapper = new ClassLoaderWrapper();
    InputStream in = classLoaderWrapper.getResourceAsStream(resource, loader);
    if (in == null) throw new IOException("Could not find resource " + resource);
    return in;
  }
 
//classLoaderWrapper.getResourceAsStream(resource, loader)
  InputStream getResourceAsStream(String resource, ClassLoader[] classLoader) {
    for (ClassLoader cl : classLoader) {
      if (null != cl) {
 
        // 关键就是这句话
        InputStream returnValue = cl.getResourceAsStream(resource);
 
        // now, some class loaders want this leading "/", so we'll add it and try again if we didn't find the resource
        if (null == returnValue) returnValue = cl.getResourceAsStream("/" + resource);
 
        if (null != returnValue) return returnValue;
      }
    }
    return null;
  }
```

 **总结1）：主要就是通过ClassLoader.getResourceAsStream()来获取指定classpath路径下的Resource**

## **new SqlSessionFactoryBuilder().build(reader)获取SessionFactory**

```
  public SqlSessionFactory build(Reader reader) {
    return build(reader, null, null);
  }
 
//build()
  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      // 主要就是这句话
      // 实现了两个功能parse.parse()解析了xml；build(configuration)创建了SqlSessionFactory
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
//核心
  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```

*** 下面看下parse.parse()方法如何进行xml解析**

```
//XMLConfigBuilder.parse()  
public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
 
// parseConfiguration()
// 可以看到主要是对xml各节点的分析，我们重点关注下mapperElement()方法
  private void parseConfiguration(XNode root) {
    try {
      propertiesElement(root.evalNode("properties")); //issue #117 read properties first
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      settingsElement(root.evalNode("settings"));
      environmentsElement(root.evalNode("environments")); // read it after objectFactory and objectWrapperFactory issue #631
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));// 重点关注下这个方法
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
 
// mapperElement(root.evalNode("mappers"))
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          // 1.获取resource信息，也就是对应的mapper.xml对应路径
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            // 2.解析对应的mapper.xml文件，
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            // 3.将解析出来的Mapper对象添加到Configuration中
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

  **可以看到，通过解析configuration.xml文件，获取其中的Environment、Setting，重要的是将<mappers>下的所有<mapper>解析出来之后添加到Configuration，Configuration类似于配置中心，所有的配置信息都在这里**



​    *** 解析完成之后，我们来看下parse(configuration)如何生成一个SQLSessionFactory**

## **我们来看下parse(configuration)如何生成一个SQLSessionFactory**

```
  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```

非常简单，直接把Configuration当做参数，直接new一个DefaultSqlSessionFactory

## **sqlSessionFactory.openSession()开启一个SqlSession**

```
  public SqlSession openSession() {
    //configuration.getDefaultExecutorType() = ExecutorType.SIMPLE;
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
  }
 
// openSessionFromDataSource()
  private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      // 1.transactionFactory默认为 ManagedTransactionFactory
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      // 2.创建一个Transaction，注意该Transaction是org.apache.ibatis.transaction.Transaction
      // Connection即从此处获取的
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
        
      // 3.创建一个Executor，默认为SimpleExecutor，参考Executor代码可知，主要的CRUD操作就是再此处完成的
      final Executor executor = configuration.newExecutor(tx, execType);
      // 4.将Executor和Configuration做为参数封装到DefaultSqlSession，默认返回该SqlSession
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```

总结3）：该方法中有许多新的类出现，下面我们集中看下这些类，简单了解下这些类的作用，以便我们更好的理解代码

## **TransactionFactory**

​    类似于我们的SessionFactory，主要是用来生产Transaction的，注意这个Transaction是org.apache.ibatis.transaction.Transaction

```
public interface TransactionFactory {
 
  void setProperties(Properties props);
  Transaction newTransaction(Connection conn);
  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);
}

```

## **org.apache.ibatis.transaction.Transaction**

可以看到getConnection()方法返回的就是我们梦寐以求的java.sql.Connection，后续的操作都是基于此的

​    并且还有一些关于事务的操作commit、rollback等

```
public interface Transaction {
  // java.sql.Connection
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
}

```

## **Executor**

 根据其接口方法可以看到CRUD操作在这里被实现，看来SqlSession将具体的操作都委托为Executor了

```
public interface Executor {
...
  int update(MappedStatement ms, Object parameter) throws SQLException;
 
  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey cacheKey, BoundSql boundSql) throws SQLException;
 
  <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
 
  Transaction getTransaction();
    ...
 
}

```

## 总结

```
 总结3：

    * 解析configuration.xml文件，生成对应的Environment、Setting、Mapper，并注册到Configuration。Configuration相当于配置管理中心，所有的配置都在这里体现

    * 生成org.apache.ibatis.transaction.Transaction实现类，里面我们需要的java.sql.Connection

    * 根据Transaction封装Executor，Executor负责主要的CRUD操作

    * 将Configuration和Executor封装到SqlSession中，SqlSession对外提供统一操作入口，内部委托为Executor来执行


```

# 剩下就是 DefaultSqlSession的方法实现了

例如:

```
SqlSession.selectOne()
SqlSession.selectList() 等具体逻辑

```



# 案例

```
public class HelloWord {
    private static SqlSessionFactory sqlSessionFactory;
    private static Reader reader;
 
    static {
        try {
            // 1.读取mybatis配置文件，并生成SQLSessionFactory
            reader = Resources.getResourceAsReader("config/Configure.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static SqlSessionFactory getSession() {
        return sqlSessionFactory;
    }
    /**
	 * @param args
	 */
    public static void main(String[] args) {
        // 2.获取session，主要的CRUD操作均在SqlSession中提供
        SqlSession session = sqlSessionFactory.openSession();
        try {
            // 3.执行查询操作
            // 该方法包括三个步骤：封装入参数、执行查询、封装结果为对象类型
            User user = (User) session.selectOne("com.yiibai.mybatis.models.UserMapper.GetUserByID", 1);
            if(user!=null){
                String userInfo = "名字："+user.getName()+", 所属部门："+user.getDept()+", 主页："+user.getWebsite();
                System.out.println(userInfo);
            }
        } finally {
            session.close();
        }
    }

```

