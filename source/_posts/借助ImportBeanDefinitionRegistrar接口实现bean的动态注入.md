---
title: 借助ImportBeanDefinitionRegistrar接口实现bean的动态注入
date: 2019-09-26 14:44:09
tags: [spring,java,动态注入]
---

# 借助ImportBeanDefinitionRegistrar接口实现bean的动态注入

### *ImportBeanDefinitionRegistrar*

#### spring官方就是用这种方式，实现了`@Component`、`@Service`等注解的动态注入机制。定义一个ImportBeanDefinitionRegistrar的实现类，然后在有`@Configuration`注解的配置类上使用`@Import`导入

#### 这里我们用一个简单的例子，我们需要实现类似`@Componet`的功能，添加了`@Mapper`注解的类会被自动加入到spring容器中。

#### 首先创建一个`@Mapper`注解，再新建一个`CountryMapper`类，使用该Mapper注解。

<!--more-->

```java
package com.linjingc.mybaitissourcecodedemo;

import org.springframework.stereotype.Service;

import java.lang.annotation.*;

@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
public @interface TestMapper {
}

@TestMapper
@Service
class CountryMapper {
}
```

#### 创建`MapperAutoConfiguredMyBatisRegistrar`实现`ImportBeanDefinitionRegistrar`，同时可以继承一些Aware接口，获得spring的一些数据

- BeanFactoryAware
- ResourceLoaderAware
- EnvironmentAware

```java
package com.linjingc.mybaitissourcecodedemo;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.io.ResourceLoader;
import org.springframework.core.type.AnnotationMetadata;

public class MapperAutoConfiguredMyBatisRegistrar
		implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, BeanFactoryAware {

    private ResourceLoader resourceLoader;

    private BeanFactory beanFactory;

    //ImportBeanDefinitionRegistrar实现
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        MyClassPathBeanDefinitionScanner scanner = new MyClassPathBeanDefinitionScanner(registry, false);
        scanner.setResourceLoader(resourceLoader);
        scanner.registerFilters();
        scanner.doScan("com.linjingc.mybaitissourcecodedemo.dao");
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }
}
```

#### 实现`registerBeanDefinitions`方法。但是有一个问题，我们并不知道需要register哪些bean。这里我们还需要借助一个类`ClassPathBeanDefinitionScanner`，也就是扫描器，通过扫描器获取我们需要注册的bean。先简单看一下spring源码中的的定义

```
/**
 * A bean definition scanner that detects bean candidates on the classpath,
 * registering corresponding bean definitions with a given registry ({@code BeanFactory}
 * or {@code ApplicationContext}).
 *
```

#### 需要继承`ClassPathBeanDefinitionScanner`，扫描使用`@Mapper`的注解的类，`ClassPathBeanDefinitionScanner`又继承`ClassPathScanningCandidateComponentProvider`类，`ClassPathScanningCandidateComponentProvider`中有两个TypeFilter集合，includeFilters、excludeFilters。满足任意includeFilters会被加载，同样的满足任意excludeFilters不会被加载。看下具体实现：

```java
package com.linjingc.mybaitissourcecodedemo;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.core.type.filter.AnnotationTypeFilter;

import java.util.Set;

@Slf4j
public class MyClassPathBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {

    public MyClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters) {
        super(registry, useDefaultFilters);
    }

    protected void registerFilters() {
        addIncludeFilter(new AnnotationTypeFilter(TestMapper.class));
    }

    @Override
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        return super.doScan(basePackages);
    }
}
```

#### 核心代码就是**registerFilters()方法**，然后在我们的`ImportBeanDefinitionRegistrar`实现类中调用

```java
	//ImportBeanDefinitionRegistrar实现
	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        MyClassPathBeanDefinitionScanner scanner = new MyClassPathBeanDefinitionScanner(registry, false);
        scanner.setResourceLoader(resourceLoader);
        scanner.registerFilters();
        scanner.doScan("com.linjingc.mybaitissourcecodedemo.dao");
    }
```

#### 测试

```java
package com.linjingc.mybaitissourcecodedemo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

    @Autowired
    ApplicationContext applicationContext;

    @Autowired
    CountryMapper countryMapper;

    @Test
    public void contextLoads() {
        System.out.println(countryMapper.getClass());
    }
}
```

#### 结果

```
class com.linjingc.mybaitissourcecodedemo.CountryMapper
```

#### 最后

#### 这里我们的CountryMapper是class类型并不是interface，要想实现Mybatis的接口注入，需要使用动态代理创建bean，这里只是一个简单的抛砖引玉。