---
title: springboot配置错误页面及全局异常
date: 2019-07-10 21:58:29
tags: [SpringBoot]
---

# springboot配置错误页面及全局异常

## spring1.x中处理方式

```java
@Bean
    public EmbeddedServletContainerCustomizer containerCustomizer() {
        return new EmbeddedServletContainerCustomizer() {
            @Override
            public void customize(ConfigurableEmbeddedServletContainer container) {
                ErrorPage error401Page = new ErrorPage(HttpStatus.UNAUTHORIZED, "/401");
                ErrorPage error405Page = new ErrorPage(HttpStatus.METHOD_NOT_ALLOWED, "/405");
                ErrorPage error404Page = new ErrorPage(HttpStatus.NOT_FOUND, "/404");
                ErrorPage error500Page = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500");

                container.addErrorPages(error401Page,error405Page, error404Page, error500Page);
            }
        };
    }
```

## spring2.x中处理方式

```java
@Component
public class ErrorConfig implements ErrorPageRegistrar {

    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        ErrorPage error400Page = new ErrorPage(HttpStatus.BAD_REQUEST, "/error400Page");
        ErrorPage error401Page = new ErrorPage(HttpStatus.UNAUTHORIZED, "/error401Page");
        ErrorPage error404Page = new ErrorPage(HttpStatus.NOT_FOUND, "/error404Page");
        ErrorPage error500Page = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error500Page");
        registry.addErrorPages(error400Page,error401Page,error404Page,error500Page);
    }
}
```

那么此时只要出现了错误，就会找到相应的 http 状态码，而后跳转到指定的错误路径上进行显示。

<!--more-->

# ErrorPageAction跳转处理

```java
@Controller
public class ErrorPageAction {
    @RequestMapping(value = "/error400Page")
    public String error400Page() {
        return "404";
    }
    @RequestMapping(value = "/error401Page")
    public String error401Page() {
        return "401";
    }
    @RequestMapping(value = "/error404Page")
    public String error404Page(Model model) {
        model.addAttribute("code","6666666");
        model.addAttribute("msg","服务器降级中......");
        return "404";
    }
    @RequestMapping(value = "/error500Page")
    public String error500Page(Model model) {
        return "500";
    }
}
```

```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
</head>
<body>
    <p th:text="${code}"/>
    <p th:text="${msg}"/>
</body>
</html>
```

# 全局异常处理

## 返回错误页面配置

如果此时配置有错误页，那么这个时候错误会统一跳转到 500 所在的路径上进行错误的显示，但是如果说现在希望能够显示 出错误更加详细的内容呢？

 所以这个时候可以单独定义一个页面进行错误的信息显示处理，而这个页面，可以定义在“src/main/view/templates/error.html”， 这个页面里面要求可以输出一些信息；

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class) // 所有的异常都是Exception子类
    public ModelAndView defaultErrorHandler(HttpServletRequest request, Exception e) { // 出现异常之后会跳转到此方法
        ModelAndView mav = new ModelAndView("error"); // 设置跳转路径
        mav.addObject("exception", e); // 将异常对象传递过去
        mav.addObject("url", request.getRequestURL()); // 获得请求的路径
        return mav;
    }
}
```

error.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>SpringBoot模版渲染</title>
    <link rel="icon" type="image/x-icon" href="/images/study.ico"/>
    <meta http-equiv="Content-Type" content="text/html;charset=UTF-8"/>
</head>
<body>
<p th:text="${url}"/>
<p th:text="${exception.message}"/>
</body>
</html>
```

## 返回Rest错误信息

```java
package cn.study.microboot.advice;

import javax.servlet.http.HttpServletRequest;

import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

//@ControllerAdvice// 作为一个控制层的切面处理
@RestControllerAdvice
public class GlobalExceptionHandler {
    public static final String DEFAULT_ERROR_VIEW = "error"; // 定义错误显示页，error.html
    @ExceptionHandler(Exception.class) // 所有的异常都是Exception子类
    public Object defaultErrorHandler(HttpServletRequest request,Exception e) {
        class ErrorInfo {
            private Integer code ;
            private String message ;
            private String url ;
            public Integer getCode() {
                return code;
            }
            public void setCode(Integer code) {
                this.code = code;
            }
            public String getMessage() {
                return message;
            }
            public void setMessage(String message) {
                this.message = message;
            }
            public String getUrl() {
                return url;
            }
            public void setUrl(String url) {
                this.url = url;
            }
        }
        ErrorInfo info = new ErrorInfo() ;
        info.setCode(100);     // 标记一个错误信息类型
        info.setMessage(e.getMessage());
        info.setUrl(request.getRequestURL().toString());
        return info ;
    }
}
```

