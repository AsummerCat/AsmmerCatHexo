---
title: SpringBoot全局异常处理@ControllerAdvice
date: 2018-11-30 10:04:27
tags: [SpringBoot]
---

# 使用@ExceptionHandler当前Controller异常处理

全局异常处理,这样就不用每次都在Controller中手动捕获异常

<!--more-->

>@ExceptionHandler可以使用在任何用@Controller注解修饰的类中，设置出现某种异常的时候执行，具体代码如下：
   
# 例子

## Contreoller 

```
package com.linjingc.aop.controller1;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author cxc
 * @date 2018/11/26 13:37
 */
@RestController
@RequestMapping("test")
public class TestController {
    @RequestMapping("/")
    public String index() {
        System.out.println("随便输出一下");
        int i = 1 / 0;
        return "hello";
    }
}

```


## Contreoller全局异常捕获@ControllerAdvice

```
package com.linjingc.aop.controller1;

import lombok.extern.log4j.Log4j2;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Controller全局异常捕获
 *
 * @author cxc
 */
@ControllerAdvice
@Log4j2
public class GlobalExceptionHandler {


    //这里表示捕获Exception.class的异常
    @ExceptionHandler(Exception.class)
    @ResponseBody
    String handleException(Exception e) {
        log.info("输出异常{}", e);
        return "Exception Deal!" + e.getMessage();
    }

    //这里表示捕获RuntimeException.class的异常
    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    String handleException1(Exception e) {
        log.info("输出异常{}", e);
        return "RuntimeException Deal!" + e.getMessage();
    }

    //这里表示捕获ArithmeticException.class的异常
    @ExceptionHandler(ArithmeticException.class)
    @ResponseBody
    String handleException2(Exception e) {
        log.info("输出异常{}", e);
        return "ArithmeticException Deal!" + e.getMessage();
    }
}
```

这里没有捕获的先后顺序 异常按照小到大去匹配

#单独Controller的异常捕获  @ExceptionHandler

上面的方式是全局的 现在写的是单个类中的异常捕获

<font color="red">需要注意的是 当全局和单独类异常捕获都存在的时候 当前类的执行顺序先于全局</font>

```
package com.linjingc.aop.controller1;

import lombok.extern.log4j.Log4j2;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author cxc
 * @date 2018/11/26 13:37
 */
@RestController
@RequestMapping("test")
@Log4j2
public class TestController {
    //这里表示捕获ArithmeticException.class的异常
    @ExceptionHandler(ArithmeticException.class)
    @ResponseBody
    String handleException2(Exception e) {
        log.info("输出异常{}", e);
        return "ArithmeticException Deal!" + e.getMessage();
    }

    @RequestMapping("/")
    public String index() {
        System.out.println("随便输出一下");
        int i = 1 / 0;
        return "hello";
    }
}
```