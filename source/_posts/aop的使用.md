---
title: aop的使用
date: 2018-11-26 21:21:07
tags: [java,Spring]
---

# 切面编程

aop

# 相关jar包

```
<!--加入spirngAOP包-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
```

# 使用
首先 我就当你已经了解了java 注解的相关知识

<!--more-->

# 自定义注解

```
package com.linjingc.aop.annotation;

import org.springframework.stereotype.Component;

import java.lang.annotation.*;

/**
 * @author cxc
 * @date 2018/11/26 13:53
 */
@Documented
@Component
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface UserAop {
    //这里需要注意  interface前面要加上@  这里表示这个interface 是一个注解
}
```

注解相关的知识在另外一篇文章有描述 可以查看一下

---

# 编写切面类

AspectJ 支持 5 种类型的通知注解:

* @Before: 前置通知, 在方法执行之前执行
* @After: 后置通知, 在方法执行之后执行 。
* @AfterRunning: 返回通知, 在方法返回结果之后执行
* @AfterThrowing: 异常通知, 在方法抛出异常之后
* @Around: 环绕通知, 围绕着方法执行

```
@Aspect
@Configuration
public class AopConfiguration {

}
```
这一样表示这个类是一个切面类

---

# 编写切面方法

```
 //声明个切面，切哪呢？切到 com.linjingc.aop.controller 这个目录下，以save开头的方法，方法参数(..)和返回类型(*)不限
    // @Pointcut("execution(* com.lxk.service.StudentService.save*(..))")
    
    //这里是表示 切到这个目录下的所有方法--并且 有带NeeLogin这个方法
   // @Pointcut("execution(public * com.biniu.yxx.controller.*.*(..)) && @annotation(com.biniu.yxx.annotation.NeedLogin)")

    @Pointcut("execution(* com.linjingc.aop.controller.*.*(..))")
    private void executeService() {
    }
```


# 获取到当前request

```
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
		ServletRequestAttributes sra = (ServletRequestAttributes) requestAttributes;
		HttpServletRequest request = sra.getRequest();
```

# before 前置通知

```
 /**
     * 前置通知
     */
    @Before("executeService()")
    public void before(JoinPoint joinPoint) {
        String s = joinPoint.toString();
        System.out.println(s);
        System.out.println("before method   start ...");

        System.out.println("before method   end ...");
    }
```

# around 环绕通知
>当定义一个Around增强处理方法时，该方法的第一个形参必须是 ProceedingJoinPoint 类型，  
>
>   在增强处理方法体内，调用ProceedingJoinPoint的proceed方法才会执行目标方法------这就是@Around增强处理可以完全控制目标方法执行时机、如何执行的关键；  
> 
>   如果程序没有调用ProceedingJoinPoint的proceed方法，则目标方法不会执行。

```
    @Around("addAdvice()")
    public Object Interceptor(ProceedingJoinPoint pjp) throws Throwable {
        RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) requestAttributes;
        HttpServletRequest request = sra.getRequest();
        Long staffId = (Long) request.getAttribute("staffId");
        if (staffId == null) {
            JsonResult jsonResult = new JsonResult();
            jsonResult.setMsg("请先登录");
            jsonResult.setStatus("-112");
            return jsonResult;
        } else {
            return pjp.proceed();
        }
    }
```

## 在around中还可以加入 改变参数

调用ProceedingJoinPoint的proceed方法时，还可以传入一个Object[ ]对象，该数组中的值将被传入目标方法作为实参。如果传入的Object[ ]数组长度与目标方法所需要的参数个数不相等，或者Object[ ]数组元素与目标方法所需参数的类型不匹配，程序就会出现异常。

`Object rvt=jp.proceed(new String[]{"被改变的参数"});`

```

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
 
@Aspect
public class AroundAdviceTest {
	
	@Around("execution(* com.bean.*.*(..))")
	public Object processTx(ProceedingJoinPoint jp) throws Throwable{
		System.out.println("执行目标方法之前，模拟开始事务...");
		Object rvt=jp.proceed(new String[]{"被改变的参数"});
		System.out.println("执行目标方法之后，模拟结束事务...");
		return rvt+"新增的内容";
	}
}

```

##  around详解

```
@Around("execution(public int lzj.com.spring.aop.ArithmeticCalculator.*(int, int))")
    public Object aroundMethod(ProceedingJoinPoint pdj){
        /*result为连接点的放回结果*/
        Object result = null;
        String methodName = pdj.getSignature().getName();

        /*前置通知方法*/
        System.out.println("前置通知方法>目标方法名：" + methodName + ",参数为：" + Arrays.asList(pdj.getArgs()));

        /*执行目标方法*/
        try {
            result = pdj.proceed();

            /*返回通知方法*/
            System.out.println("返回通知方法>目标方法名" + methodName + ",返回结果为：" + result);
        } catch (Throwable e) {
            /*异常通知方法*/
            System.out.println("异常通知方法>目标方法名" + methodName + ",异常为：" + e);
        }

        /*后置通知*/
        System.out.println("后置通知方法>目标方法名" + methodName);

        return result;
    }
}
```

#  AfterReturning 返回通知

```
 /**
     * 被切入后 目标执行完毕后 必定会执行
     *
     * @param joinPoint
     */
    @AfterReturning("executeService()")
    public void after(JoinPoint joinPoint) {
        System.out.println("After method   start ...");

        System.out.println("After method   end ...");
    }
```
    
# AfterThrowing 异常通知
  没有@Around，则 要代理的方法执行 异常才会被@AfterThrowing捕获；
  
 ```
 /*通过throwing属性指定连接点方法出现异常信息存储在ex变量中，在异常通知方法中就可以从ex变量中获取异常信息了*/
@AfterThrowing(value="execution(public int lzj.com.spring.aop.ArithmeticCalculator.*(int, int))",
            throwing="ex")
    public void afterReturning(JoinPoint point, Exception ex){
        String methodName = point.getSignature().getName();
        List<Object> args = Arrays.asList(point.getArgs());
        System.out.println("连接点方法为：" + methodName + ",参数为：" + args + ",异常为：" + ex);
    }

 ```
# 关于JoinPoint的方法和简单注释。

```
package org.aspectj.lang;
 
import org.aspectj.lang.reflect.SourceLocation;
 
public interface JoinPoint {
    String toString();         //连接点所在位置的相关信息
    String toShortString();     //连接点所在位置的简短相关信息
    String toLongString();     //连接点所在位置的全部相关信息
    Object getThis();         //返回AOP代理对象
    Object getTarget();       //返回目标对象
    Object[] getArgs();       //返回被通知方法参数列表
    Signature getSignature();  //返回当前连接点签名
    SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置
    String getKind();        //连接点类型
    StaticPart getStaticPart(); //返回连接点静态部分
}

```  
 
 
    