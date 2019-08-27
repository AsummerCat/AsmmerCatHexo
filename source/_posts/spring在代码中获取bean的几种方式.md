---
title: spring在代码中获取bean的几种方式
date: 2019-08-27 21:33:11
tags: [spring,java,SpringBoot]
---

# spring在代码中获取bean的几种方式

# 第一种

```java
@Service
public class BeanFactoryHelper implements BeanFactoryAware {
    
    private static BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }
    
    public static Object getBean(String beanName){

　　　　 if(beanFactory == null){
            throw new NullPointerException("BeanFactory is null!");
        }
　　　　 return beanFactory.getBean(beanName); 
　　} 
}
```

<!--more-->

# 第二种

实现ApplicationContextAware接口，代码也很简单：

```java
@Service
public class ApplicationContextHelper implements ApplicationContextAware {
    
    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    
    public static Object getBean(String beanName){
        if(applicationContext == null){
            throw new NullPointerException("ApplicationContext is null!");
        }
        return applicationContext.getBean(beanName);
    }

}
```

# 第三种

效率低于上面两种

```java
WebApplicationContext webApplication = ContextLoader.getCurrentWebApplicationContext();   
helloService = (HelloService) webApplication.getBean("helloService");  
logger.info(helloService.processService("ContextLoader.getCurrentWebApplicationContext() test"));  
```

# 测试时候使用

上面三种方法，只有容器启动的时候，才会把BeanFactory和ApplicationContext注入到自定义的helper类中，如果在本地junit测试的时候，如果需要根据bean的名称获取bean对象，则可以通过ClassPathXmlApplicationContext来获取一个ApplicationContext，代码如下：

```java
@Test
    public void test() throws SQLException {
        //通过从classpath中加载spring-mybatis.xml实现bean的获取
        ApplicationContext context = new ClassPathXmlApplicationContext("spring-mybatis.xml");
        IUserService userService = (IUserService) context.getBean("userService");

        User user = new User();
        user.setName("test");
        user.setAge(20);
        userService.addUser(user);
    }
```

