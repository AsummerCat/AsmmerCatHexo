---
title: SpringMVC源码解析之初始化流程一
date: 2019-09-10 23:41:30
tags: [SpringMvc,源码解析]
---

# SpringMVC源码解析之初始化流程(一)

https://blog.csdn.net/gududedabai/article/details/83375156

https://www.liangzl.com/get-article-detail-126117.html 

# 初始化流程

初始化方法->DispatcherServlet 执行onRefresh()

## onRefresh() 

```java
@Override
 protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
  }
```

```java
	protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
  }
```

这里就是初始化handlerMapping 适配器 等等的初始化过程

## initHandlerMappings

![initHandlerMappings初始化](/img/2019-09-03/16.png)

获取出 三个HandlerMapping

<!--more-->

# 现在来看下启动流程

## HttpServletBean ->执行初始化方法init() 

*让子类做任何他们喜欢的初始化*

![](/img/2019-09-03/17.png)

 initServletBean() 这个方法是由其子类 FrameworkServlet 实现，因此， 接下来 FramworkServlet 会执行 initServletBean 这个方法，下面就继续看看 initServletBean 方法源码

## FrameworkServlet->  initServletBean()

![](/img/2019-09-03/18.png)

可以看到 initServletBean 方法中就调用了一个 initFrameworkServlet 方法和 initWebApplicationContext 方法，其中initFrameworkServlet方法是由子类实现

## FrameworkServlet->  initWebApplicationContext ()

```java
 protected WebApplicationContext initWebApplicationContext() {
 
        //此处的 rootContext 在你配置了ContextLoaderListener的时候注入的
        //通过分析ContextLoaderListenr的源码，可以看到
        //ContextLoaderListener通过ContextLoader根据ApplicationContext.xml的配置会创建一个xmlWebApplicationContext
        //如果没有配置ContextLoaderListener,本处将为null,但不影响springMVC,为何？通过接下来的分析，就能看到原因
        WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
        WebApplicationContext wac = null;
       
       //当webApplicationContext已经存在，那么就直接使用，使用之前会先设置rootContext,为其跟。
       //配置完成之后refresh一次，refresh会涉及到IOC的内容，本处不做探讨。
 
        if (this.webApplicationContext != null) {
            wac = this.webApplicationContext;
            if (wac instanceof ConfigurableWebApplicationContext) {
                ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
                if (!cwac.isActive()) {
                    if (cwac.getParent() == null) {
                        cwac.setParent(rootContext);
                    }
 
                    this.configureAndRefreshWebApplicationContext(cwac);
                }
            }
        }
        //如果不存在webApplicationContext,那么先去ServletContext中查找
        if (wac == null) {
            wac = this.findWebApplicationContext();
        }
        //如果上述没有查到，那么就创建webApplicationContext
        if (wac == null) {
            wac = this.createWebApplicationContext(rootContext);
        }
 
        if (!this.refreshEventReceived) {
             //此方法由DispatcherServlet调用
            this.onRefresh(wac);
        }
        //将webApplicationContext保存在ServletContext
        if (this.publishContext) {
            //将上下文发布为servlet上下文属性。
            String attrName = this.getServletContextAttributeName();
            this.getServletContext().setAttribute(attrName, wac);
            if (this.logger.isDebugEnabled()) {
                this.logger.debug("Published WebApplicationContext of servlet '" + this.getServletName() + "' as ServletContext attribute with name [" + attrName + "]");
            }
        }
 
        return wac;
    }
```



## DispatcherServlet->onRefresh()  实现父类方法

这样 初始化流程基本就完成了