---
title: 面试相关-Spring
date: 2020-02-04 00:30:18
tags: [面试相关]
---

# Spring

### bean的作用域

```
single 默认 单例
prototype 原型  每次调用getbean初始化
request 每次请求创建一个bean
session 在一次会话中共享一个bean
```

##Spring IOC 启动过程

```
refresh方法
第一步读取bean配置信息

第二步 解析bean的定义
BeanFactory
第三步根据bean注册表实例化Bean   

第四步将Bean实例放到容器中

第五步使用Bean
```

## bean的生命周期

<!--more-->

### 简单版本

```
实例化Bean->设置对象属性（依赖注入)->处理Aware接口(感知自己的属性)->BeanPostProcessor(自定义处理)
->初始化 ->使用 ->销毁
```



```
当一个bean实例化时可能需要执行一些初始化动作进入使bean达到一个可用的状态。同样,当不再需要bean时，将bean从容器中移除,可能需要销毁。
```

```
1.Bean容器找到配置文件中Spring Bean的定义。

2.Bean容器利用Java Reflection API创建一个Bean的实例。

3.如果涉及到一些属性值，利用set()方法设置一些属性值。

4.如果Bean实现了BeanNameAware接口，调用setBeanName()方法，传入Bean的名字。

5.如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。

6.如果Bean实现了BeanFactoryAware接口，调用setBeanClassFacotory()方法，传入ClassLoader对象的实例。

7.与上面的类似，如果实现了其他*Aware接口，就调用相应的方法。

8.如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessBeforeInitialization()方法。

9.如果Bean实现了InitializingBean接口，执行afeterPropertiesSet()方法。

10.如果Bean在配置文件中的定义包含init-method属性，执行指定的方法。

11.如果有和加载这个Bean的Spring容器相关的BeanPostProcess对象，执行postProcessAfterInitialization()方法。

12.当要销毁Bean的时候，如果Bean实现了DisposableBean接口，执行destroy()方法。

13.当要销毁Bean的时候，如果Bean在配置文件中的定义包含destroy-method属性，执行指定的方法。
```





## 事务相关

### 事务类型

```
声明式事务: @Transactional 滕莎克血肉  
       1.基于XML的声明式事务
       2.基于注解的声明式事务
编程式事务:
```



### 事务传级别7种

```
PROPAGATION_REQUIRED：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
PROPAGATION_SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。
PROPAGATION_MANDATORY：使用当前的事务，如果当前没有事务，就抛出异常。
PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。
PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。
```

常用:

```
PROPAGATION_REQUIRED 默认事务
PROPAGATION_REQUIRES_NEW  挂起当前事务 新建事务
PROPAGATION_NEVER  不使用事务
```

<!--more-->

### 传播级别

```
传播级别
isolation.REPEATABLE_READ 可重复读默认级别 mysql
isolation.READ_COMMITTED 读已提交 ORCLE默认级别
```





# 真题

### **Spring AOP和AspectJ AOP有什么区别？**

```
AOP是运行时Spring AOP是属于运行时增强，而AspectJ是编译时增强。
Spring AOP基于代理（Proxying），而AspectJ基于字节码操作（Bytecode Manipulation）。
```

## Spring 中的单例 bean 的线程安全问题了解吗？

```
大部分时候我们并没有在系统中使用多线程，所以很少有人会关注这个问题。单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1.在Bean对象中尽量避免定义可变的成员变量（不太现实）。

2.在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。
```

## Spring 框架中用到了哪些设计模式？

```
1.工厂设计模式 : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。

2.代理设计模式 : Spring AOP 功能的实现。

3.单例设计模式 : Spring 中的 Bean 默认都是单例的。

4.模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。

5.包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

6.观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。

7.适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。
```

## bean的循环引用如何处理

```
1. 拆开依赖 在代码中 引入依赖

2.任意对象使用懒加载 @lazy
```

