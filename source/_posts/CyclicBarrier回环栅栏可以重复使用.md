---
title: CyclicBarrier回环栅栏可以重复使用
date: 2018-11-05 13:54:30
tags: 多线程
---

# CyclicBarrier

字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，`CyclicBarrier`可以被重用。我们暂且把这个状态就叫做barrier，当调用`await`()方法之后，线程就处于barrier了。

<!--more-->

<font color="red">与`countDownLatch`差别  

* 1. countDownLatch只能初始化一次
* 2. CyclicBarrier 可以重复使用
* 3. countDownLatch 方法:
*  记数: countDown();    等待:await();
* 4. CyclicBarrier 方法:
*  等待:await();  到达数量后就直接运行
</font>


```
1）CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：

CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；

而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；

另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。
```

# 构造与方法

## 构造

`CyclicBarrier`类位于`java.util.concurrent`包下，`CyclicBarrier`提供2个构造器:  

```
public CyclicBarrier(int parties, Runnable barrierAction) {
}
 
public CyclicBarrier(int parties) {
}
```

## 方法 

然后CyclicBarrier中最重要的方法就是await方法，它有2个重载版本：
第一个版本比较常用，用来挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务；

第二个版本是让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务。

```
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```


# 例子

```
package threadtest.cyclicbarrier;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * @author cxc
 * @date 2018/11/1 22:55
 * CyclicBarrier线程测试类     等待
 */
public class CyclicBarrierThread implements Runnable {
    private CyclicBarrier cyclicBarrier;
    private String name;

    public CyclicBarrierThread(CyclicBarrier cyclicBarrier, String name) {
        this.cyclicBarrier = cyclicBarrier;
        this.name = name;
    }

    @Override
    public void run() {
        try {
            System.out.println("我准备好了->" + name);
            //等待其他执行完毕通知运行
            System.out.println("我开始等待了->" + name);
            cyclicBarrier.await();
            System.out.println("我们出发吧->" + name);
        } catch (BrokenBarrierException e) {
            e.getLocalizedMessage();
        } catch (InterruptedException e) {
            e.getLocalizedMessage();
        }
    }
}
```

```
package threadtest.cyclicbarrier;

import threadtest.countdownlatchtest.CountDownLatchThread;

import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author cxc
 * @date 2018/11/5 14:07
 * CyclicBarrier 回环栅栏
 */
public class CyclicBarrierTest {
    static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);

    public static void main(String[] args) {
        //小明和小强在等待5个人的车子开出去后出发   这边使用 await() 等待
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(new CyclicBarrierThread(cyclicBarrier, "小明"));
        executorService.execute(new CyclicBarrierThread(cyclicBarrier, "小强"));
        executorService.execute(new CyclicBarrierThread(cyclicBarrier, "小1"));
        executorService.execute(new CyclicBarrierThread(cyclicBarrier, "小2"));
        executorService.execute(new CyclicBarrierThread(cyclicBarrier, "小3"));
        //executorService.execute(new CyclicBarrierThread(cyclicBarrier, "小4"));
    }
}
```