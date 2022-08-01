---
title: springboot跨大版本升级到2.6.7问题处理
date: 2022-08-01 13:50:40
tags: [SpringBoot]
---
# springboot跨大版本升级到2.6.7问题处理

## 1.nacos无法使用 临时解决方案
2022年4月27日  
由于目前nacos没有更新版本推上线,导致sptingboot2.3版本之后无法使用 缺少`ConfigurationBeanFactoryMetadata`

`maven引入`
```
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>nacos-config-spring-boot-starter</artifactId>
            <version>0.2.7</version>
        </dependency>
```
<!--more-->
`application.yml加入配置`
```
nacos:
  config:
    username: nacos
    password: nacos
    type: yaml
    server-addr: nacos.com:8848
    namespace: f4d7b5af-f4a4-4c09-af99-968c8ee07d16
    data-id: application.yml
    auto-refresh: true
    bootstrap:
      enable: true

# 还需要开启spring的可重载Bean
spring:
  main:
    allow-bean-definition-overriding: true
      
```
`新增两个配置类 `
1.NacosBootConfigurationPropertiesBinderExtend
```

import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.config.annotation.NacosConfigurationProperties;
import com.alibaba.nacos.spring.context.properties.config.NacosConfigurationPropertiesBinder;
import com.alibaba.nacos.spring.core.env.NacosPropertySource;
import com.alibaba.nacos.spring.util.ObjectUtils;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.boot.context.properties.bind.Bindable;
import org.springframework.boot.context.properties.bind.Binder;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.ResolvableType;
import org.springframework.core.env.StandardEnvironment;

import java.lang.reflect.Method;

public class NacosBootConfigurationPropertiesBinderExtend extends NacosConfigurationPropertiesBinder {
    private final ConfigurableApplicationContext applicationContext;

    private final StandardEnvironment environment = new StandardEnvironment();

    public NacosBootConfigurationPropertiesBinderExtend(
            ConfigurableApplicationContext applicationContext) {
        super(applicationContext);
        this.applicationContext = applicationContext;
    }

    @Override
    protected void doBind(Object bean, String beanName, String dataId, String groupId,
                          String configType, NacosConfigurationProperties properties, String content,
                          ConfigService configService) {
        synchronized (this) {
            String name = "nacos-bootstrap-" + beanName;
            NacosPropertySource propertySource = new NacosPropertySource(dataId, groupId, name, content, configType);
            environment.getPropertySources().addLast(propertySource);
            ObjectUtils.cleanMapOrCollectionField(bean);
            Binder binder = Binder.get(environment);
            ResolvableType type = getBeanType(bean, beanName);
            Bindable<?> target = Bindable.of(type).withExistingValue(bean);
            binder.bind(properties.prefix(), target);
            publishBoundEvent(bean, beanName, dataId, groupId, properties, content, configService);
            publishMetadataEvent(bean, beanName, dataId, groupId, properties);
            environment.getPropertySources().remove(name);
        }
    }

    private ResolvableType getBeanType(Object bean, String beanName) {
        Method factoryMethod = findFactoryMethod(beanName);
        if (factoryMethod != null) {
            return ResolvableType.forMethodReturnType(factoryMethod);
        }
        return ResolvableType.forClass(bean.getClass());
    }

    public Method findFactoryMethod(String beanName) {
        ConfigurableListableBeanFactory beanFactory = this.applicationContext.getBeanFactory();
        if (beanFactory.containsBeanDefinition(beanName)) {
            BeanDefinition beanDefinition = beanFactory.getMergedBeanDefinition(beanName);
            if (beanDefinition instanceof RootBeanDefinition) {
                return ((RootBeanDefinition) beanDefinition).getResolvedFactoryMethod();
            }
        }
        return null;
    }
}
```
2.NacosConfigBootBeanDefinitionRegistrarExtend
```
import com.alibaba.boot.nacos.config.autoconfigure.NacosConfigBootBeanDefinitionRegistrar;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.context.annotation.Primary;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.stereotype.Component;

/**
 * @description 解决springboot2.6.5版本中没有ConfigurationBeanFactoryMetadata 导致老版本nacos无法使用问题
 */
@Primary
@Component
public class NacosConfigBootBeanDefinitionRegistrarExtend extends NacosConfigBootBeanDefinitionRegistrar {

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        DefaultListableBeanFactory defaultListableBeanFactory = (DefaultListableBeanFactory) beanFactory;
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder
                .rootBeanDefinition(NacosBootConfigurationPropertiesBinderExtend.class);
        defaultListableBeanFactory.registerBeanDefinition(
                NacosBootConfigurationPropertiesBinderExtend.BEAN_NAME,
                beanDefinitionBuilder.getBeanDefinition());
    }

    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
                                        BeanDefinitionRegistry registry) {

    }
}
```


# sagger版本未更新无法使用,输出doc.html的内容为空
解决方案: 手动注入
`pom引入`
```
<dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>
```
`新增注入类`
```
/**
     * 解决springboot 2.6.7版本后的空指针问题
     *
     * @return
     */
    @Bean
    public static BeanPostProcessor springfoxHandlerProviderBeanPostProcessor() {
        return new BeanPostProcessor() {

            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider) {
                    customizeSpringfoxHandlerMappings(getHandlerMappings(bean));
                }
                return bean;
            }

            private <T extends RequestMappingInfoHandlerMapping> void customizeSpringfoxHandlerMappings(List<T> mappings) {
                List<T> copy = mappings.stream()
                        .filter(mapping -> mapping.getPatternParser() == null)
                        .collect(Collectors.toList());
                mappings.clear();
                mappings.addAll(copy);
            }

            @SuppressWarnings("unchecked")
            private List<RequestMappingInfoHandlerMapping> getHandlerMappings(Object bean) {
                try {
                    Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
                    field.setAccessible(true);
                    return (List<RequestMappingInfoHandlerMapping>) field.get(bean);
                } catch (IllegalArgumentException | IllegalAccessException e) {
                    throw new IllegalStateException(e);
                }
            }
        };
    }

```
`application.yml新增配置`
```
spring:
  main:
    allow-circular-references: true
  mvc:
    pathmatch:
      matching-strategy: ANT_PATH_MATCHER    
```