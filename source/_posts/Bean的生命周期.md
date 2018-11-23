---
title: Bean的生命周期
date: 2018-11-21 16:47:19
tags: Spring
---

# 生命周期回调
开发者通过实现Spring的`InitializeingBean`和`DisposableBean`接口，就可以让容器来管理Bean的生命周期。容器会在调用`afterPropertiesSet`()之后和destroy()之前会允许Bean在初始化和销毁Bean的时候执行一些操作。

>JSR-250的@PostConstruct和@PreDestroy注解就是现代Spring应用生命周期回调的最佳实践。使用这些注解意味着Bean不会再耦合在Spring特定的接口上。详细内容，后续将会介绍。 如果开发者不想使用JSR-250的注解，仍然可以考虑使用init-method和destroy-method的定义来解耦Spring接口。

<!--more-->

# 注解的使用 初始化->销毁

```
@PostConstruct注解 初始化

@PreDestroy注解 销毁
```

# 初始化回调

org.springframework.beans.factory.InitializingBean接口允许Bean在所有的必要的依赖配置配置完成后来执行初始化Bean的操作。InitializingBean接口中特指了一个方法：

`void afterPropertiesSet() throws Exception;`

Spring团队是建议开发者不要使用InitializingBean接口的，因为这样会将代码耦合到Spring的特定接口之上。而通过使用@PostConstruct注解或者指定一个POJO的实现方法，会比实现接口要更好。在基于XML的配置元数据上，开发者可以使用init-method属性来指定一个没有参数的方法。使用Java配置的开发者可以使用@Bean之中的initMethod属性，比如如下：

```
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```
public class ExampleBean {

    public void init() {
        // do some initialization work
    }

}
```

# 销毁回调

实现了org.springframework.beans.factory.DisposableBean接口的Bean就能通让容器通过回调来销毁Bean所引用的资源。DisposableBean接口包含了一个方法：

```
void destroy() throws Exception;
```

同InitializingBean相类似，Spring团队仍然不建议开发者来实现DisposableBean回调接口，因为这样会将开发者的代码耦合到Spring代码上。换种方式，比如使用@PreDestroy注解或者指定一个Bean支持的配置方法，比如在基于XML的配置元数据中，开发者可以在Bean标签上指定destroy-method属性。而在基于Java配置中，开发者也可以配置@Bean的destroyMethod来实现销毁回调。

```
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }

}
```

---

# Combining lifecycle mechanisms

在Spring 2.5之后，开发者有三种选择来控制Bean的生命周期行为：   
 
*  InitializingBean和DisposableBean回调接口   (no)
*  自定义的init()以及destroy方法  
*  使用@PostConstruct以及@PreDestroy注解  


开发者也可以在Bean上联合这些机制一起使用

>如果Bean配置了多个生命周期机制，而且每个机制配置了不同的方法名字，那么每个配置的方法会按照后面描述的顺序来执行。然而，如果配置了相同的名字，比如说初始化回调为init()，在不止一个生命周期机制配置为这个方法的情况下，这个方法只会执行一次。


如果一个Bean配置了多个生命周期机制，并且含有不同的方法名，执行的顺序如下：  

* 包含@PostConstruct注解的方法
* 在InitializingBean接口中的afterPropertiesSet()方法
* 自定义的init()方法


销毁方法的执行顺序和初始化的执行顺序相同：

* 包含@PreDestroy注解的方法
* 在DisposableBean接口中的destroy()方法
* 自定义的destroy()方法




