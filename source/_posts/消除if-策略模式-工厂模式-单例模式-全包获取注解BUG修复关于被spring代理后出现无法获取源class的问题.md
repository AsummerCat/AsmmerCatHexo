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

<!--more-->

这样获取到的bean 如果要是被spring利用cglib代理或者jdk代理后 获取到的object 就是代理类

代理类中无法获取到注解

然后会产生无法获取注解 并且获取的getname 也是代理类的名称

无法使用

# 修复方案

利用AopTargetUtils工具类获取源class

```java
	for (Object bean : beans.values()) {
				try {
					//获取源class
					Object target = AopTargetUtils.getTarget(bean);
					indexBiMap.put(target.getClass().getAnnotation(IndexBi.class).value(), lowerFirst(target.getClass().getSimpleName()));
				} catch (Exception e) {
					//错误
					logger.error("获取注解失败");
				}
			}
			IndexBiFactory.setIndexBiMap(indexBiMap);
```



## 工具类

```
package com.mdt.util;

import org.springframework.aop.framework.AdvisedSupport;
import org.springframework.aop.framework.AopProxy;
import org.springframework.aop.support.AopUtils;

import java.lang.reflect.Field;

/**
 * 获取源class
 * @date 2019年9月4日10:48:47
 * @author cxc
 */
public class AopTargetUtils {
	/**
	 * 获取 目标对象
	 *
	 * @param proxy 代理对象
	 * @return
	 * @throws Exception
	 */
	public static Object getTarget(Object proxy) throws Exception {

		if (!AopUtils.isAopProxy(proxy)) {
			return proxy;//不是代理对象
		}
		if (AopUtils.isJdkDynamicProxy(proxy)) {
			return getJdkDynamicProxyTargetObject(proxy);
		} else { //cglib
			return getCglibProxyTargetObject(proxy);
		}
	}


	private static Object getCglibProxyTargetObject(Object proxy) throws Exception {
		Field h = proxy.getClass().getDeclaredField("CGLIB$CALLBACK_0");
		h.setAccessible(true);
		Object dynamicAdvisedInterceptor = h.get(proxy);

		Field advised = dynamicAdvisedInterceptor.getClass().getDeclaredField("advised");
		advised.setAccessible(true);

		Object target = ((AdvisedSupport) advised.get(dynamicAdvisedInterceptor)).getTargetSource().getTarget();

		return target;
	}


	private static Object getJdkDynamicProxyTargetObject(Object proxy) throws Exception {
		Field h = proxy.getClass().getSuperclass().getDeclaredField("h");
		h.setAccessible(true);
		AopProxy aopProxy = (AopProxy) h.get(proxy);

		Field advised = aopProxy.getClass().getDeclaredField("advised");
		advised.setAccessible(true);

		Object target = ((AdvisedSupport) advised.get(aopProxy)).getTargetSource().getTarget();

		return target;
	}


}

```

