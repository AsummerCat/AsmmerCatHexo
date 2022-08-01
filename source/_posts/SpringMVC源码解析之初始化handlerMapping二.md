---
title: SpringMVC源码解析之初始化handlerMapping二
date: 2019-09-10 23:44:30
tags: [SpringMvc,源码解析]
---

# SpringMVC源码解析之初始化handlerMapping二



我们根据工作机制中三部分来分析 Spring MVC 的源代码.

1. ApplicationContext初始化时建立所有url和controller类的对应关系(用Map保存);
2. 根据请求url找到对应的controller,并从controller中找到处理请求的方法;
3. request参数绑定到方法的形参,执行方法处理请求,并返回结果视图.

<!--more-->

# ApplicationObjectSupport的setApplicationContext方法.(spring包下)

setApplicationContext方法中核心部分就是初始化容器initApplicationContext(context),子类AbstractDetectingUrlHandlerMapping实现了该方法,所以我们直接看子类中的初始化容器方法.

![](/img/2019-09-03/19.png)

![](/img/2019-09-03/20.png)

## 初始化容器具体实现 AbstractDetectingUrlHandlerMapping->initApplicationContext();(mvc包下)

![](/img/2019-09-03/21.png)

![](/img/2019-09-03/22.png)

## 初始化过程中获取bean和url关系,子类实现determineUrlsForHandler(beanName); 方法

继承 AbstractDetectingUrlHandlerMapping->

获取每个controller中的url,不同的子类有不同的实现,这是一个典型的模板设计模式.

```
	protected abstract String[] determineUrlsForHandler(String beanName);
```

![](/img/2019-09-03/23.png)

```
determineUrlsForHandler(String beanName)方法的作用是获取每个controller中的url,不同的子类有不同的实现,这是一个典型的模板设计模式.

因为开发中用的最多的就是用注解来配置controller中的url。DefaultAnnotationHandlerMapping是AbstractDetectingUrlHandlerMapping的子类,处理注解形式的url映射.

所以我们这里以DefaultAnnotationHandlerMapping来进行分析.我们看DefaultAnnotationHandlerMapping是如何查beanName上所有映射的url.
```

### DefaultAnnotationHandlerMapping 从注解获取bean对应url

```
/**
     * 获取controller中所有的url
     */
    protected String[] determineUrlsForHandler(String beanName) {
        // 获取ApplicationContext容器
        ApplicationContext context = getApplicationContext();

        //从容器中获取controller
        Class handlerType = context.getType(beanName);

        // 获取controller上的@RequestMapping注解
        RequestMapping mapping = context.findAnnotationOnBean(beanName, RequestMapping.class);
        if (mapping != null) { // controller上有注解
            this.cachedMappings.put(handlerType, mapping);

            // 返回结果集
            Set<String> urls = new LinkedHashSet<String>();

            // controller的映射url
            String[] typeLevelPatterns = mapping.value();
            if (typeLevelPatterns.length > 0) { // url>0

                // 获取controller中所有方法及方法的映射url
                String[] methodLevelPatterns = determineUrlsForHandlerMethods(handlerType, true);
                for (String typeLevelPattern : typeLevelPatterns) {
                    if (!typeLevelPattern.startsWith("/")) {
                        typeLevelPattern = "/" + typeLevelPattern;
                    }
                    boolean hasEmptyMethodLevelMappings = false;
                    for (String methodLevelPattern : methodLevelPatterns) {
                        if (methodLevelPattern == null) {
                            hasEmptyMethodLevelMappings = true;
                        } else {
                            // controller的映射url+方法映射的url
                            String combinedPattern = getPathMatcher().combine(typeLevelPattern, methodLevelPattern);
                            // 保存到set集合中
                            addUrlsForPath(urls, combinedPattern);
                        }
                    }
                    if (hasEmptyMethodLevelMappings || org.springframework.web.servlet.mvc.Controller.class.isAssignableFrom(handlerType)) {
                        addUrlsForPath(urls, typeLevelPattern);
                    }
                }
                // 以数组形式返回controller上的所有url
                return StringUtils.toStringArray(urls);
            } else {
                // controller上的@RequestMapping映射url为空串,直接找方法的映射url
                return determineUrlsForHandlerMethods(handlerType, false);
            }
        } // controller上没@RequestMapping注解
        else if (AnnotationUtils.findAnnotation(handlerType, Controller.class) != null) {
            // 获取controller中方法上的映射url
            return determineUrlsForHandlerMethods(handlerType, false);
        } else {
            return null;
        }
    }
```

到这里HandlerMapping组件就已经建立所有url和controller的对应关系。

剩下就是DispatcherServlet 初始化过程了