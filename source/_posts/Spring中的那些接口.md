---
title: Spring中的那些接口
date: 2020-04-15 13:40:28
tags: [java,spring,源码解析]
---

# Spring中的那些接口

Spring中提供了很多的接口来实现Bean的扩展 和注册

## Bean的初始化及销毁方法

### 初始化实现方法 @PostConstruct

```
1、通过java提供的@PostConstruct注解；

2、通过实现spring提供的InitializingBean接口，并重写其afterPropertiesSet方法；

3、通过spring的xml bean配置或bean注解指定初始化方法，如下面实例的initMethod方法通过@bean注解指定。
```

<!--more-->

### 销毁时实现方法  @PreDestroy

```
1、通过java提供的@PreDestroy注释；

2、通过实现spring提供的DisposableBean接口，并重写其destroy方法；

3、通过spring的xml bean配置或bean注解指定销毁方法，如下面实例的destroyMethod方法通过@bean注解指定。

```

```java
    
    public User(){
		System.out.println("User 被实例化");
	}
	
    /**
	 * 自定义的初始化的方法
	 */
	@PostConstruct
	public void postConstruct(){
		System.out.println("...postConstruct...");
	}
	
	/**
	 * 销毁前回调方法
	 */
	@PreDestroy
	public  void preDestory(){
		System.out.println("--preDestory---");
	}
	/**
	 * 销毁前的回调方法
	 */
	public void end(){
		System.out.println("--end--");
	}
```

### 执行顺序

```java
User 被实例化
...postConstruct...
--自定义的初始化的方法--
--preDestory---
--end--
```

### InitializingBean接口 初始化使用

该接口的作用是：允许一个bean在它的所有必须属性被BeanFactory设置后，来执行初始化的工作，该接口中只有一个方法，afterPropertiesSet

```java
public interface InitializingBean {
	void afterPropertiesSet() throws Exception;
}
```

### DisposableBean接口 销毁时候使用

该接口的作用是：允许在容器销毁该bean的时候获得一次回调。DisposableBean接口也只规定了一个方法：destroy

```java
public interface DisposableBean {
	void destroy() throws Exception;
}
```

如果bean实现了InitializingBean接口，会自动调用afterPropertiesSet方法，在bean被销毁的时候如果实现了DisposableBean接口会自动回调destroy方法后然后再销毁

## Aware接口

Aware接口从字面上翻译过来是感知捕获的含义。单纯的bean（未实现Aware系列接口）是没有知觉的；实现了Aware系列接口的bean可以访问Spring容器。这些Aware系列接口增强了Spring bean的功能，但是也会造成对Spring框架的绑定，增大了与Spring框架的耦合度。（Aware是“意识到的，察觉到的”的意思，实现了Aware系列接口表明：可以意识到、可以察觉到）

### ApplicationContextAware

```java
void setApplicationContext(ApplicationContext applicationContext)
```

### BeanClassLoaderAware

```java
void setBeanClassLoader(ClassLoader classLoader);
```

### BeanFactoryAware

```java
void setBeanFactory(BeanFactory beanFactory)
```

### BeanNameAware

```java
void setBeanName(String name);
```

### BeanPostProcessor接口

```
1.初始化之前 如果bean实现了BeanPostProcessor接口，它的postProcessBeforeInitialization方法将被调用；
2.初始化之后 它的postProcessAfterInitialization接口方法将被调用；
```



Aware系列接口，主要用于辅助Spring bean访问Spring容器

### 测试

```java
@Component
public class TestAware implements ApplicationContextAware, BeanClassLoaderAware, BeanFactoryAware, BeanNameAware {
	private ApplicationContext ac;
	private ClassLoader cl;
	private BeanFactory bf;
	private String beanName;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		ac = applicationContext;
	}

	@Override
	public void setBeanClassLoader(ClassLoader classLoader) {
		cl = classLoader;
	}


	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
		bf = beanFactory;
	}

	@Override
	public void setBeanName(String name) {
		beanName = name;
	}
```

```JAVA
public class MainStart {
	public static void main(String[] args) {
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
		TestAware testAware = applicationContext.getBean(TestAware.class);
		System.out.println("输出Application:" + testAware.getAc());
		System.out.println("输出beanFactory:" + testAware.getBf());
		System.out.println("输出beanName:" + testAware.getBeanName());
		System.out.println("输出classLoader:" + testAware.getCl());
	}
}
```

### 输出

```
输出Application: org.springframework.context.annotation.AnnotationConfigApplicationContext@5a07e868
输出beanFactory:org.springframework.beans.factory.support.DefaultListableBeanFactory@531d72c
输出beanName:testAware
输出classLoader:sun.misc.Launcher$AppClassLoader@18b4aac2
```

## BeanPostProcessor后置处理器

后置处理器是全局的 也就是每个Bean初始化的时候都会调用这个

### postProcessBeforeInitialization()调用前调用

对于每个bean后置处理器来说，它的postProcessBeforeInitialization方法会在每个bean的初始化之前被调用一次；

### postProcessAfterInitialization初始化后调用

对于每个bean后置处理器来说，它的postProcessAfterInitialization方法会在每个bean的初始化之后被调用一次；

### 多个BeanPostProcessor执行执行顺序

```java
implements  Ordered

@Override
	public int getOrder() {
		// TODO Auto-generated method stub
		return 10;
	}
```



### 测试

```java
@Component
public class TestBeanPostProcessor implements BeanPostProcessor, InitializingBean {

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("初始化之前");
		System.out.println(bean);
		return null;
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("初始化之后");
           //这里需要注意如果一个返回为null 后续的所有后置处理器就不执行了
		return null;
	}
	}
```

### 输出

```java
实例化
初始化之前
com.linjingc.config.AppConfig$$EnhancerBySpringCGLIB$$7861b138@22a71081
初始化之后
```

## InstantiationAwareBeanPostProcessor接口

```java
package org.springframework.beans.factory.config;
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

	Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException;

	boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException;

	PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException;

}
```

InstantiationAwareBeanPostProcessor继承于BeanPostProcessor 除了父类的两个初始化方法 还添加了3个自己的方法

### postProcessBeforeInstantiation 最先执行的方法 实例化之前调用

### postProcessAfterInstantiation 实例化之后调用 ->还未设置属性

### postProcessPropertyValues 对属性值进行修改

```java
@Component
public class TestInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {
	@Override
	public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		System.out.println(beanName+"实例化之前");
        //这里需要注意如果一个返回不为null 后续的所有后置处理器就不执行了
		return null;
	}

	@Override
	public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		System.out.println(beanName+"实例化之后");
		return false;
	}

	@Override
	public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {
		return null;
	}
}

```

### 输出

```
person实例化之前    postProcessBeforeInstantiation
person实例化之后    postProcessAfterInstantiation
person初始化之前    postProcessBeforeInitialization  初始化这两个是父类调用的方法
person初始化之后    postProcessAfterInitialization
```

### InstantiationAwareBeanPostProcessor和BeanPostProcessor差别

```
InstantiationAwareBeanPostProcessor里面 方法后缀是 Instantiation  表示实例化,对象还未生成

BeanPostProcessor里面 方法后缀 Initialization    表示初始化,对象已经生成

```

## BeanFactoryPostProcessor接口 

Spring IoC容器允许BeanFactoryPostProcessor在容器实例化任何bean之前读取bean的定义(配置元数据)，并可以修改它。同时可以定义BeanFactoryPostProcessor，通过设置’order’属性来确定各个BeanFactoryPostProcessor执行顺序。
   注册一个BeanFactoryPostProcessor实例需要定义一个Java类来实现BeanFactoryPostProcessor接口，并重写该接口的postProcessorBeanFactory方法。通过beanFactory可以获取bean的定义信息，并可以修改bean的定义信息。

### 简单来说 就是在实例化之前 可以修改bean定义的数据(切勿getBean进行实例化操作)

### 测试

```
public class TestBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		List<String> beanNames = Stream.of(beanFactory.getBeanDefinitionNames()).collect(Collectors.toList());
		String name = beanNames.stream().filter(e -> e.equals("person")).collect(Collectors.joining());

		//从BeanFactory获取到Bean定义
		BeanDefinition beanDefinition = beanFactory.getBeanDefinition(name);
		//获取属性
		MutablePropertyValues propertyValues = beanDefinition.getPropertyValues();
		// MutablePropertyValues如果设置了相关属性，可以修改，如果没有设置则可以添加相关属性信息
		propertyValues.add("name", "1111");
		System.out.println("修改了属性信息");
	}
}
```

```
需要注意的是 这个部分需要 使用@import注册到容器中使用
```

## BeanDefinitionRegistryPostProcessor(将指定的类注册到spring容器中)

继承了BeanFactoryPostProcessor接口

```java
//根据名称和类型获取bean
            BeanDefinitionRegistryPostProcessor pp = beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class);
            //把已经调用过postProcessBeanDefinitionRegistry方法的bean全部放在registryPostProcessors中
            registryPostProcessors.add(pp);
            //把已经调用过postProcessBeanDefinitionRegistry方法的bean的名称全部放在processedBeans中
            processedBeans.add(ppName);
            //执行此bean的postProcessBeanDefinitionRegistry方法
            pp.postProcessBeanDefinitionRegistry(registry);

```



## BeanFactory接口

```
这个是Spring核心接口之一
BeanFactory接口是Spring容器的核心接口，负责：实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。
```

## FactoryBean接口

需要实现FactoryBean

```java
@Component
public class TestFactoryBean implements FactoryBean {
	//注入属性
	private String name;

	@Override
	public Object getObject() throws Exception {
		System.out.println("工厂类实例化前");
		Person pp = new Person(name);
		System.out.println("工厂类实例化后");
		return  pp;
	}

	@Override
	public Class<?> getObjectType() {
		return Person.class;
	}

	@Override
	public boolean isSingleton() {
		return true;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}

```

### 工厂Bean生产两个对象

```java
@Bean
	public TestFactoryBean xiaoming(){
		TestFactoryBean testFactoryBean = new TestFactoryBean();
		testFactoryBean.setName("小明");
		return testFactoryBean;
	}
	@Bean
	public TestFactoryBean xiaodong(){
		TestFactoryBean testFactoryBean = new TestFactoryBean();
		testFactoryBean.setName("小东");
		return testFactoryBean;
	}
```

### 测试

获取到被工厂类生产的对象

```java
		AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
//		Person person = (Person) applicationContext.getBean("person");
		//获取工厂类生产的对象
		Person person1 = (Person) applicationContext.getBean("xiaoming");
		Person person2 = (Person) applicationContext.getBean("xiaodong");
		//获取工厂类 需要加上&
		String simpleName = applicationContext.getBean("&testFactoryBean").getClass().getSimpleName();
		System.out.println(person1.toString());
		System.out.println(person2.toString());
```

### 主要用处是

```
1.可以实现代理原生类 进行扩展的功能
2.可以实现动态加载  比如载入数据源之类的
```

## ApplicationListener监听器

目的为了用来监听 容器注销 容器初始化Refresh完成之后的事件 进行操作

### 注入方式

````
1. 使用@Component 注册bean

2. 使用@EventListene 使用EventListenerMethodProcessor处理器来解析方法上的@EventListener；
````



```java
语法:
 ApplicationListener<ContextRefreshedEvent>
```

### 事件

```
部分事件
ContextClosedEvent        关闭容器触发   常用  applicationContext.close();
ContextRefreshedEvent     初始化或者刷新操作的时候触发  常用 applicationContext.refresh();
ContextStartedEvent       启动触发的事件    applicationContext.start(); 
ContextStoppedEvent       停止时触发的事件   applicationContext.stop();
```

```java
发布一个事件：applicationContext.publishEvent()；
```

### 测试

```java
/**
 * Application初始化或者刷新操作的监听器
 */
@Component
public class RefreshListener implements ApplicationListener<ContextRefreshedEvent> {

	@Override
	public void onApplicationEvent(ContextRefreshedEvent event) {
		ApplicationContext ac = event.getApplicationContext();
		// 根容器为Spring容器
		//在整合SpringMVC的时候需要判断一下 因为存在父子容器
		if (ac.getParent() == null) {
			System.out.println("RefreshListener-> 监听到初始化或者刷新事件");
		}
	}
}
```



