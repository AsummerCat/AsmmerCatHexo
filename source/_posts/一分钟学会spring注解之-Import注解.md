---
title: 一分钟学会spring注解之@Import注解
date: 2018-11-29 16:10:58
tags: Spring
---

# @Import注解是什么

通过导入的方式实现 把实例导入到Spring容器中

<!--more-->

# @Import的三种使用方式
在4.2之前只支持导入配置类  
在4.2，@Import注解支持导入普通的java类,并将其声明成一个bean

通过查看@Import源码可以发现@Import注解只能注解在类上，以及唯一的参数value上可以配置3种类型的值Configuration，ImportSelector，ImportBeanDefinitionRegistrar，源码如下：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    /**
     * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
     * or regular component classes to import.
     */
    Class<?>[] value();
}
```

# 需要实例化的类

```
public class DemoService {
    public void doSomething(){
        System.out.println("ok");
    }
 
}
```

# 配置类 导入

```
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
 
@Configuration
@Import(DemoService.class)//在spring 4.2之前是不不支持的
public class DemoConfig {
 
}
```

# 测试

```
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
 
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("com..example");
        DemoService ds = context.getBean(DemoService.class);
        ds.doSomething();
    }
 
}
```

# 结果

```
ok
```