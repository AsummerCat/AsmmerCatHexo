---
title: Spring的多级缓存处理之循环依赖的问题
date: 2020-04-08 21:34:38
tags: [Spring,循环依赖]
---

# Spring的多级缓存处理->循环依赖的问题

## spring中解决的问题

```
spring默认是开启循环依赖的
不能解决由于构造方法导致的循环依赖问题

可以解决 set注入 
```

## 构造注入解决方案

### 1.使用 @Lazy 加入到循环依赖的两个构造方法中 让其中一个进行懒加载

```
 public Foo(@Lazy Bar bar) {
          this.bar = bar;
      }
```

### 2. 使用 @PostConstruct  拆开循环依赖 让代码执行处理

```
@Component
public class CircularDependencyA {
 
    @Autowired
    private CircularDependencyB circB;
 
    @PostConstruct
    public void init() {
        circB.setCircA(this);
    }
 
    public CircularDependencyB getCircB() {
        return circB;
    }
}
```

<!--more-->

```
@Component
public class CircularDependencyB {
 
    private CircularDependencyA circA;
     
    private String message = "Hi!";
 
    public void setCircA(CircularDependencyA circA) {
        this.circA = circA;
    }
     
    public String getMessage() {
        return message;
    }
}
```

### 3.修改为其他方式注入 比如set注入

## 流程

循环依赖的问题


```
在 A的bean生成的过程中调用了B对象

B对象的构建中调用了A
这样就是一个典型的循环依赖
```


解决思路:

```
1.A的Bean的构建过程中标记为正在构建的状态
2.A构建调用了B的构造方法 ->生成B的Bean
3.B的Bean生成过程中 调用A 
4.尝试从AbstractBeanFactory.getSingleton(beanName)中获取A 
也就是尝试从缓存中获取对象
步骤如下:
5.判断单例池(第一次缓存的位置)中是否存在A的对象 
this.singletonObjects.get(beanName);
存在直接返回对象 
6.不存在&&判断A是否标记为构建中->
earlySingletonObjects.get(beanName)获取出对象,获取到直接返回对象
->提前暴光的单例对象的Cache 。【用于检测循环引用，与singletonFactories互斥】

7.获取不到->
singletonFactories.get(beanName) 获取出对象
如果获取到对象的话
	this.earlySingletonObjects.put(beanName,singletonObject);
	this.singletonFactories.remove(beanName);
添加二级缓存(这边就是单独用来保证循环依赖的),
清除三级缓存(doCreateBean中出现,也就是初始化Bean的过程)

这样保证了高可用性,因为构建对象的可能会有多个后置处理器来代理生成对象,这样做了一次缓存后就不用每次都进行判断生成对象
```

## Spring默认是支持循环依赖的


```
allowEarlyReference默认是true

DefaultSingletonBeanRegistry类的
Object getSingleton(String beanName, boolean allowEarlyReference) 方法

可以手动修改Spring的allowEarlyReference来实现不支持循环依赖
```


## 缓存清除的部分


```
DefaultSingletonBeanRegistry.addSingleton()
1. 在加入单例池中后
2. 会移除三级缓存二级缓存
3. 添加记录表保存已经处理的bean


protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
			//加入单例池
			this.singletonObjects.put(beanName, singletonObject);
			//从三级缓存移除(针对的不是处理循环依赖的)
			this.singletonFactories.remove(beanName);
			//从二级缓存中移除(循环依赖的时候,早期对象存在于二级缓存)
			this.earlySingletonObjects.remove(beanName);
			//用来记录保存已经处理的bean
			this.registeredSingletons.add(beanName);
		}
	}
```