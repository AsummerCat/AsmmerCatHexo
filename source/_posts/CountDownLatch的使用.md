---
title: CountDownLatch的使用
date: 2018-11-01 22:42:09
tags: 多线程
---

# CountDownLatch 用法

CountDownLatch典型用法1：某一线程在开始运行前等待n个线程执行完毕。将CountDownLatch的计数器初始化为n <font color="red">new CountDownLatch(n)</font> ，每当一个任务线程执行完毕，就将计数器减1 <font color="red">countdownlatch.countDown()</font>，当计数器的值变为0时，在CountDownLatch上 <font color="red">await()</font> 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

CountDownLatch典型用法2：实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的<font color="red">CountDownLatch(1)</font>，将其计数器初始化为1，多个线程在开始执行任务前首先 <font color="red">coundownlatch.await()</font>，当主线程调用 <font color="red">countDown()</font> 时，计数器变为0，多个线程同时被唤醒。

<!--more-->

# CountDownLatch的不足
CountDownLatch是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。

# 主要方法

```
//创建有初始值的记数阀门 
CountDownLatch countDownLatch = new CountDownLatch(5)

//记数减1
countDownLatch.countDown();

//进入等待触发
countDownLatch.await();

//直到countDown() 执行到 0的时候 等待的才继续执行  
 (也就是达到预期后触发,  countDown次数达到->初始值的数量)

```

# 例子

>   小明和小强两个await() 在等待其他5个人开车开走后 他们才出发

## 先创造两个线程类

```
package threadtest.countdownlatchtest;

import java.util.concurrent.CountDownLatch;

/**
 * @author cxc
 * @date 2018/11/1 22:55
 * CountDownLatch线程测试类     等待
 */
public class CountDownLatchThread implements Runnable {
    private CountDownLatch countDownLatch;
    private String name;

    public CountDownLatchThread(CountDownLatch countDownLatch, String name) {
        this.countDownLatch = countDownLatch;
        this.name = name;
    }

    @Override
    public void run() {
        try {
            System.out.println("我准备好了->" + name);
            //等待其他执行完毕通知运行
            System.out.println("我开始等待了->" + name);
            countDownLatch.await();
            System.out.println("我们出发吧->" + name);
        } catch (InterruptedException e) {
            e.getLocalizedMessage();
        }
    }
}
```
```
package threadtest.countdownlatchtest;

import java.util.concurrent.CountDownLatch;

/**
 * @author cxc
 * @date 2018/11/1 22:55
 *  CountDownLatch线程测试类     直接执行
 */
public class CountDownLatchThread1 implements Runnable{
    private CountDownLatch countDownLatch;
    private String name;

    public CountDownLatchThread1(CountDownLatch countDownLatch, String name) {
        this.countDownLatch = countDownLatch;
        this.name = name;
    }

    @Override
    public void run() {
            System.out.println(name+"的车子开出去了");
            //执行成功 减1
             countDownLatch.countDown();
    }
}

```

## 主方法

```
package threadtest.countdownlatchtest;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author cxc
 * @date 2018/11/1 22:54
 * CountDownLatch测试  (等待集合 然后执行)
 */
public class CountDownLatchTest {
    //创建一个有初始值的记数阀门
    static final CountDownLatch countDownLatch = new CountDownLatch(5);

    public static void main(String[] args) {
        //小明和小强在等待5个人的车子开出去后出发   这边使用 await() 等待
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(new CountDownLatchThread(countDownLatch, "小明"));
        executorService.execute(new CountDownLatchThread(countDownLatch, "小强"));

        //这边 countDown 执行
        executorService.execute(new CountDownLatchThread1(countDownLatch, "小西"));
        executorService.execute(new CountDownLatchThread1(countDownLatch, "小黑"));
        executorService.execute(new CountDownLatchThread1(countDownLatch, "小白"));
        executorService.execute(new CountDownLatchThread1(countDownLatch, "小绿"));
        executorService.execute(new CountDownLatchThread1(countDownLatch, "小美"));

    }
}
```

## 预期结果

```
我准备好了->小明
我开始等待了->小明
我准备好了->小强
我开始等待了->小强
小西的车子开出去了
小白的车子开出去了
小黑的车子开出去了
小美的车子开出去了
小绿的车子开出去了
我们出发吧->小明
我们出发吧->小强

Process finished with exit code 0

```