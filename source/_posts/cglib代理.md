---
title: cglib代理
date: 2018-11-16 16:28:52
tags: 代理工厂
---

# 什么是CGLIB?

CGLIB是一个功能强大，高性能的代码生成包。它为没有实现接口的类提供代理，为JDK的动态代理提供了很好的补充。通常可以使用Java的动态代理创建代理，但当要代理的类没有实现接口或者为了更好的性能，CGLIB是一个好的选择。

<!--more-->

# CGLIB原理

## CGLIB原理：
动态生成一个要代理类的子类，子类重写要代理的类的所有不是final的方法。在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。它比使用java反射的JDK动态代理要快。

##底层： 
使用字节码处理框架ASM，来转换字节码并生成新的类。不鼓励直接使用ASM，因为它要求你必须对JVM内部结构包括class文件的格式和指令集都很熟悉。

## CGLIB缺点：
对于final方法，无法进行代理。

# CGLIB的应用

广泛的被许多AOP的框架使用，例如Spring AOP和dynaop。Hibernate使用CGLIB来代理单端single-ended(多对一和一对一)关联。

# 使用CGLib实现

使用CGLib实现动态代理，完全不受代理类必须实现接口的限制，而且CGLib底层采用ASM字节码生成框架，使用字节码技术生成代理类，比使用Java反射效率要高。唯一需要注意的是，CGLib不能对声明为final的方法进行代理，因为CGLib原理是动态生成被代理类的子类。

## 被代理类

```
package proxytest.cglibproxytest;

/**
 * 目标对象,没有实现任何接口
 */
public class UserDao {

    public void save() {
        System.out.println("----已经保存数据!----");
    }
}
```

## 代理类 1

```
package proxytest.cglibproxytest;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * Cglib子类代理工厂
 * 对UserDao在内存中动态构建一个子类对象
 *
 * @author cxc
 * @date 2018/11/16 16:55
 */
public class CglibProxyTest implements MethodInterceptor {
    //被代理的类
    private Object target;

    public CglibProxyTest() {
    }

    public CglibProxyTest(Object target) {
        this.target = target;
    }

    //给目标对象创建一个代理对象
    public Object getProxyInstance() {
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(target.getClass());
        //3.设置回调函数
        en.setCallback(this);
        //4.创建子类(代理对象)
        return en.create();

    }

    /**
     * 重写方法拦截在方法前和方法后加入业务
     * Object obj为目标对象
     * Method method为目标方法
     * Object[] params 为参数，
     * MethodProxy proxy CGlib方法代理对象
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib代理开始");
//执行目标对象的方法
        Object returnValue = method.invoke(target, objects);
        System.out.println("cglib代理执行结束");
        return returnValue;
    }
}

```

## 代理类 2

```
package proxytest.cglibproxytest;

import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * Cglib子类代理工厂
 * 对UserDao在内存中动态构建一个子类对象
 *
 * @author cxc
 * @date 2018/11/16 16:55
 */
public class CglibProxyTest2 implements MethodInterceptor {

    /**
     * 重写方法拦截在方法前和方法后加入业务
     * Object obj为目标对象
     * Method method为目标方法
     * Object[] params 为参数，
     * MethodProxy proxy CGlib方法代理对象
     */
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("cglib代理开始");
//执行目标对象的方法
        Object returnValue = methodProxy.invokeSuper(o, objects);
        System.out.println("cglib代理执行结束");
        return returnValue;
    }
}

```

## 执行 

```
package proxytest.cglibproxytest;

import org.springframework.cglib.proxy.Enhancer;

/**
 * Cglib子类代理工厂
 * 对UserDao在内存中动态构建一个子类对象
 *
 * @author cxc
 * @date 2018/11/16 16:55
 */
public class CglibProxyMain {
    public static void main(String[] args) {
        //两种实现方式

        //方式一
        test1();
        //方式二
        test2();
    }


    private static void test1() {
        //目标对象
        UserDao target = new UserDao();
        //代理对象
        UserDao proxy = (UserDao) new CglibProxyTest(target).getProxyInstance();

        proxy.save();
    }

    private static void test2() {
        //1.工具类
        Enhancer en = new Enhancer();
        //2.设置父类
        en.setSuperclass(UserDao.class);
        //3.设置回调函数
        en.setCallback(new CglibProxyTest2());
        //4.创建子类(代理对象)
        UserDao target = (UserDao) en.create();

        target.save();
    }
}

```


# 注意

```
在Spring的AOP编程中:
如果加入容器的目标对象有实现接口,用JDK代理
如果目标对象没有实现接口,用Cglib代理
```