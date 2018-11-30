---
title: Spring实例化容Config方式
date: 2018-11-21 14:45:22
tags: Spring
---

# 实例化容器
使用AnnotationConfigApplicationContext实例化Spring容器


@Configuration类本身作为bean被注册了，并且类内所有声明的@Bean方法也被作为bean注册了。当@Component和JSR-330类作为输入时，它们被注册为bean，并且被假设如@Autowired或@Inject的DI元数据在类中需要的地方使用。

<!--more-->

---

# 简单构造


与使用Spring XML配置作为输入实例化ClassPathXmlApplicationContext过程类似，当实例化AnnotationConfigApplicationContext时@Configuration类可能作为输入。这就允许在Spring容器中完全可以不使用XML：


```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

# 还可以构建@Component 实例化bean
正如上面所提到的，AnnotationConfigApplicationContext不仅仅局限于和@Configuration类合作。任意@Component或JSR-330注解的类都可以作为构造方法的输入。比如：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```