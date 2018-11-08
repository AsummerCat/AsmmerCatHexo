---
title: Volatile共享变量和ThreadLocal线程本地变量
date: 2018-11-04 19:28:40
tags: 多线程
---

# 参考
* [volatile、ThreadLocal、synchronized等3个关键字区别](https://blog.csdn.net/paincupid/article/details/47346423)  

* [Synchronized、lock、volatile、ThreadLocal、原子性总结、Condition](https://blog.csdn.net/sinat_29621543/article/details/78065062)

<!--more-->


# Volatile

同步变量  主内存不会有副本 操作直接刷新住内存

<font color="red">volatile主要是用来在多线程中同步变量。当一个变量被volatile修饰后，该变量就不能被缓存到线程的内存中</font>  


例子:

```
package threadtest.volatiletest;

import java.util.concurrent.TimeUnit;

/**
 * @author cxc
 * @date 2018/11/5 13:00
 */
public class VolatileDemo {
    private static  boolean flag = false;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                while (!flag) {
                }
            }
        };

        t1.start();
        TimeUnit.SECONDS.sleep(1);
        flag = true;
    }

}
```

```
package threadtest.volatiletest;

import java.util.concurrent.TimeUnit;

/**
 * @author cxc
 * @date 2018/11/5 13:00
 * 使用volatile程序会终止
 */
public class VolatileDemo {
    private static boolean flag = false;
    //如果不使用volatile 程序不会终止 会有一个副本
    //private static  boolean flag = false;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread() {
            @Override
            public void run() {
                while (!flag) {
                }
            }
        };

        t1.start();
        TimeUnit.SECONDS.sleep(1);
        flag = true;
    }

}
```
加上volatile后线程t1在大约1s后停止了，而不加volatile线程t1不会停止，原因在上面已论述。所以volatile的一个作用是可以禁止指令重排序。

---


# ThreadLocal 

首先ThreadLocal和本地线程没有一毛钱关系，更不是一个特殊的<font color="red">Thread，它只是一个线程的局部变量(其实就是一个Map),ThreadLocal会为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。</font>

```
package threadtest.threadlocaltest;

/**
 * @author cxc
 * @date 2018/11/5 13:17
 */
public class ThreadLocalDemo {
    private static ThreadLocal<Integer> threadLocal=new ThreadLocal<>();
    private static int num=1;

    public static void main(String[] args){
        for (int i = 0; i < 100; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println("添加第一个:"+num);
                    threadLocal.set(num+1);
                    System.out.println("测试第一个"+threadLocal.get());
                    threadLocal.remove();
                }
            }).start();
        }
    }
}

```

ThreadLocal是一个类，利用它的set、get方法可以在不同线程中都去使用同一个对象的副本。如果多个线程希望使用相同的初始值，但是又不希望在不同线程中相互干扰，这时可以使用ThreadLocal类。