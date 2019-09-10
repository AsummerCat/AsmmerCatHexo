---
title: 消除if-策略模式+工厂模式+单例模式-全包获取注解BUG修复关于被spring代理后出现无法获取源class的问题
date: 2019-09-10 23:39:29
tags: [java,设计模式]
---

# 消除if-策略模式+工厂模式+单例模式-全包获取注解(BUG修复)关于被spring代理后出现无法获取源class的问题

在原先的版本中

获取注解的方式是这样的:

```java
@Component
public class ContextRefreshedListener implements ApplicationListener<ContextRefreshedEvent> {
    @Autowired
    private IndexBiFactory biFactory;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        HashMap<String, String> indexBiMap = new HashMap<>();
        // 根容器为Spring容器
        if (event.getApplicationContext().getParent() == null) {
            Map<String, Object> beans = event.getApplicationContext().getBeansWithAnnotation(IndexBi.class);
            for (Object bean : beans.values()) {
                bean.getClass().getAnnotation(IndexBi.class).value();
                indexBiMap.put(bean.getClass().getAnnotation(IndexBi.class).value(), bean.getClass().getName());
            }
            biFactory.setIndexBiMap(indexBiMap);
        }
    }
}
```

<!--more-->

这样获取到的bean 如果要是被spring利用cglib代理或者jdk代理后 获取到的object 就是代理类

代理类中无法获取到注解

然后会产生无法获取注解 并且获取的getname 也是代理类的名称

无法使用

# 修复方案

利用工具类获取源class