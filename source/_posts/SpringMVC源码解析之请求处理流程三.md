---
title: SpringMVC源码解析之请求处理流程三
date: 2019-09-10 23:45:43
tags: [SpringMvc,源码解析]
---

# SpringMVC源码解析之请求处理流程(三)

![执行总流程](/img/2019-09-03/1.png)



## 执行流程

### DispatcherServlet 前端控制器

因为从流程图看，用户的请求最先到达就是DispatcherServlet。他是springmvc的核心，也是中央出处理器。因此我们分析源码，先看看他是什么样的流程：通过源码可看到：他是继承FrameworkServlet，它也是springmvc提供的类，继续往下继承关系看，FrameworkServlet继承HttpServletBean，她依旧是spring提供的.最终直到他继承HttpServlet

而这个类他就是servlet。因此既然是Servlet类，那么他有一个最终的方法，就是service()方法，他是serlet最核心的方法。

<!--more-->

### service方法  拿到不同的请求方式然后处理不同的业务

因此，我们在HttpServletBean类中找service方法，发现没有，我们继续往上一层FrameworkServlet类中找，发现找到了，因此spring实现该方法在这个类去实现的。
![service初始方法](/img/2019-09-03/2.png)

这里职责主要是先拿到一个请求，然后又做了一个判断请求方式。发现不是PATCH方式就去调用父类（HttpServlet）中service()方法。他去掉用父类中的service方法其实就是去调用该类中doPost(),doGet()方法，拿到不同的请求方式然后处理不同的业务。比如以FrameworkServlet的get方式为例
![doGet请求](/img/2019-09-03/3.png)

## processRequest()执行doService() 实现 

当这个方法拿到之后，他就去调用里面的方法processRequest();该方法中一些代码我们可以不用细看，主要是跟控制器有关的代码。因此重点关注processRequest中的 doService(request, response);->跳转到DispatcherServlet类的doService(request, response);实现

### doDispatch() 具体执行逻辑

在这个方法中，依旧如上，主要代码是 this.doDispatch(request, response);，这个方法，由此可看，代码到这里，还没有进入核心区域。然后我们进入，这个方法。才算正式进入springMNV的最核心代码区域：如下

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
      ModelAndView mv = null;
      Exception dispatchException = null;

      try {
        processedRequest = checkMultipart(request);
        multipartRequestParsed = (processedRequest != request);

        // Determine handler for the current request.
        mappedHandler = getHandler(processedRequest);
        if (mappedHandler == null || mappedHandler.getHandler() == null) {
          noHandlerFound(processedRequest, response);
          return;
        }

        // Determine handler adapter for the current request.
        HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

        // Process last-modified header, if supported by the handler.
        String method = request.getMethod();
        boolean isGet = "GET".equals(method);
        if (isGet || "HEAD".equals(method)) {
          long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (logger.isDebugEnabled()) {
            logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
          }
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
           return;
           }
        }

        if (!mappedHandler.applyPreHandle(processedRequest, response)) {
           return;
        }

        try {
          // Actually invoke the handler.
          mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
        }
        finally {
          if (asyncManager.isConcurrentHandlingStarted()) {
            return;
          }
        }

        applyDefaultViewName(request, mv);
        mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
        dispatchException = ex;
      }
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Error err) {
        triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
    }
    finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
        // Instead of postHandle and afterCompletion
        mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
        return;
      }
      // Clean up any resources used by a multipart request.
      if (multipartRequestParsed) {
        cleanupMultipart(processedRequest);
      }
    }
  }
```

# doDispatch()下的->getHandler()  获取具体的hander处理器

handlerMapping->getHandler

这个Handler其实就是我们的控制器，包括我们写Controller和拦截器

由源码可得：他首先主要是创建一个视图对象 ModelAndView mv = null;然后检查当前请求是否是二进制的请求processedRequest = this.checkMultipart(request);然后就是只要代码 

 `mappedHandler = this.getHandler(processedRequest);`

![获取具体的handler](/img/2019-09-03/4.png)

由流程图可知，发送清求到控制器，控制器第二个节点就是发送第二个请求就是去拿Handler，因此可知这里才是最核心代码。由图可知他取Handler最终要去找HandlerMapping,然后他再去拿一个Handler。那么为什么要去找HandlerMapping去要一个Handler呢？首先我们在配置控制器的时候有两种方式

- 1.xml方式
- 2.注解的方式

因此spring源码他给我们不止一种控制器 。因为两种方式控制器 。因此spring并不知道我们使用的事哪一种控制器。因为两种控制器，spring去底层去找的控制的实现方式是不一样的。因此这就是为什么第二步他要去找Handler（控制器）的了。但是Handler怎么找的到呢?就是通过HandlerMapping这样一个处理器映射器。如代码可知他首先是判断当前HandlerMappers是否为空：this.handlerMappings

- 执行流程: 通过遍历handlerMappings 获取具体的控制器 判断hander 如果当前HandlerMapping不存在该内容则 跳转到下一个

#### 遍历到 HandlerMapping后 再根据每个HandlerMapping的getHandler(request)去解析知否存在该handler

![获取具体的handler](/img/2019-09-03/5.png)

![获取具体的handler](/img/2019-09-03/6.png)

这边可以看到获取到handler内容包括 5个拦截器 + 一个controller

获取成功 然后返回中央处理器。

### 获取适配器 **getHandlerAdapter**

![获取具体的Adapter](/img/2019-09-03/7.png)

获取控制器的适配器。也就是我们之前拿到了控制器，接下来要去执行控制器，也就是拿到控制器适配器中执行控制器。这里为什么要获取适配器呢？因为跟控制器映射器（也就是配置方式）一样。你就有不同的适配器。因此适配器也不是一个。跟我们上面Handler原理一样。

![获取具体的Adapter](/img/2019-09-03/8.png)

![获取具体的Adapter](/img/2019-09-03/9.png)

#### 再经过缓存相关处理

![缓存相关处理](/img/2019-09-03/9.png)

#### Adapter判断拦截器并处理->applyPreHandle

![Adapter判断拦截器并处理](/img/2019-09-03/10.png)

![Adapter判断拦截器并处理](/img/2019-09-03/11.png)

这里面就是一堆拦截器。

处理没问题  继续下一步

### 适配器执行处理器 Adapter去执行Handler

![适配器执行处理器 Adapter去执行Handler](/img/2019-09-03/11.png)

如果你有ModelAndView，就返回一个ModelAndView.然后返回给试图对象，然后把视图对象交给视图解析器，去渲染，最后响应给用户。



# 总结

因此总结，spring提供了两种HandlerMapping以及三种HandlerAdapter.他们运行匹配的关系如图：

![总结](/img/2019-09-03/13.png)

那么运行时他怎么能找到这些呢？spring是怎么配置提供的呢？

其实他们在spring配置文件就已经配置好了，当springMVC初始化时就加载实例化，获取这些对象。他们是被配置在spring的SpringwebMVC架包的servlet架包中的DispatcherServlet.properties配置文件中
![总结](/img/2019-09-03/14.png)

![总结](/img/2019-09-03/15.png)

也是在spring初始化时把控制器处理器与适配器提供好。以及把Controller在对应的自定义的Controller对象名在控制器处理器中携带着，它被放在一个map中（路径为key,对象名为value）；然后程序去遍历控制器处理器，通过请求路径去找到对应的处理器获取其中的Controller对象名称,最后与拦截器一起封装在HandlerExecutionChain一起返回

其控制器原理大致可以理解为：第一步是去找那个控制器，第二步是去执行控制器，然后返回给试图对象，然后把视图对象交给视图解析器，去渲染，最后响应给用户。