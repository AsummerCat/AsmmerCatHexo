---
title: Myabtis源码解析-加载顺序一
date: 2019-09-26 15:35:42
tags: [mybatis源码解析]
---

# mybatis 加载顺序

# 一

1、加载配置文件，解析配置文件，MyBatis基于XML配置文件创建Configuration对象的过程

2、SqlSessionFactoryBuilder根据传入的数据流生成Configuration对象，然后根据Configuration对象创建默认的SqlSessionFactory实例。创建SqlSessionFactoryBean，生产出来sqlSession，

3、SqlSession对象完成和数据库的交互
a.使用传统的MyBatis提供的API；
 这是传统的传递Statement Id 和查询参数给 SqlSession 对象，使用 SqlSession对象完成和数据库的交互；
b. 使用Mapper接口
 MyBatis 将配置文件中的每一个<mapper> 节点抽象为一个 Mapper 接口，而这个接口中声明的方法和跟<mapper> 节点中的<select|update|delete|insert> 节点项对应

4、Executor作为Mybatis的核心器，负责sql动态语句的生成和查询缓存的维护
StatementHandler 封装了JDBC Statement操作，负责对JDBC statement 的操作，如设置参数等
ParameterHandler 负责对用户传递的参数转换成JDBC Statement 所对应的数据类型
ResultSetHandler 负责将JDBC返回的ResultSet结果集对象转换成List类型的集合
TypeHandler 负责java数据类型和jdbc数据类型(也可以说是数据表列类型)之间的映射和转换
MappedStatement  MappedStatement维护一条<select|update|delete|insert>节点的封装

5、Executor依靠MappedStatement  和数据库进行交互

<!--more-->

# 二

1、加载配置文件(数据源，以及映射文件)，解析配置文件，生成Configuration，MapperedStatement
2、通过使用Configuration对象，创建sqlSessionFactory，用来生成SqlSeesion
3、sqlSession通过调用api或者mapper接口传入statementId找到对应的MapperedStatement，来调用执行sql
4、通过Executor核心器，负责sql动态语句的生成和查询缓存的维护，来进行sql的参数转换，动态sql的拼接，生成Statement对象
5、借助于MapperedStatement来访问数据库，它里面封装了sql语句的相关信息，以及返回结果信息

(1)加载配置：配置来源于两个地方，一处是配置文件，一处是Java代码的注解，将SQL的配置信息加载成为一个个MappedStatement对象（包括了传入参数映射配置、执行的SQL语句、结果映射配置），存储在内存中。
(2)SQL解析：当API接口层接收到调用请求时，会接收到传入SQL的ID和传入对象（可以是Map、JavaBean或者基本数据类型），Mybatis会根据SQL的ID找到对应的MappedStatement，然后根据传入参数对象对MappedStatement进行解析，解析后可以得到最终要执行的SQL语句和参数。
(3) SQL执行：将最终得到的SQL和参数拿到数据库进行执行，得到操作数据库的结果。
(4)结果映射：将操作数据库的结果按照映射的配置进行转换，可以转换成HashMap、JavaBean或者基本数据类型，并将最终结果返回。

(A)根据SQL的ID查找对应的MappedStatement对象。
(B)根据传入参数对象解析MappedStatement对象，得到最终要执行的SQL和执行传入参数。
(C)获取数据库连接，根据得到的最终SQL语句和执行传入参数到数据库执行，并得到执行结果。
(D)根据MappedStatement对象中的结果映射配置对得到的执行结果进行转换处理，并得到最终的处理结果。
(E)释放连接资源。

MyBATIS中有三种executor:
SimpleExecutor -- SIMPLE 就是普通的执行器。
ReuseExecutor -执行器会重用预处理语句（prepared statements）
BatchExecutor --它是批量执行器
这些就是mybatis的三种执行器。你可以通过配置文件的settings里面的元素defaultExecutorType，配置它，默认是采用SimpleExecutor如果你在Spring运用