---
title: Semaphore信号量的使用
date: 2018-11-01 13:25:43
tags: 多线程
---

# 信号量 
## Semaphore共享锁的使用
信号量(Semaphore)，又被称为信号灯，在多线程环境下用于协调各个线程, 以保证它们能够正确、合理的使用公共资源。信号量维护了一个许可集，我们在初始化Semaphore时需要为这个许可集传入一个数量值，该数量值代表同一时间能访问共享资源的线程数量。线程可以通过acquire()方法获取到一个许可，然后对共享资源进行操作，注意如果许可集已分配完了，那么线程将进入等待状态，直到其他线程释放许可才有机会再获取许可，线程释放一个许可通过release()方法完成。

# 概念
* 公平锁/非公平锁（多线程执行顺序的维度）  
* 概念理解  
* 公平锁：加锁前先查看是否有排队等待的线程，有的话优先处理排在前面的线程，先来先得。  
* 非公平锁：线程加锁时直接尝试获取锁，获取不到就自动到队尾等待。

<!--more--> 

# 方法介绍

```
(常用)
acquire() 获取一个许可
release() 线程释放一个许可

其他方法:

//构造方法摘要
//创建具有给定的许可数和非公平的公平设置的Semaphore。
Semaphore(int permits) 

//创建具有给定的许可数和给定的公平设置的Semaphore，true即为公平锁     
Semaphore(int permits, boolean fair) 

//从此信号量中获取许可，不可中断
void acquireUninterruptibly() 

//返回此信号量中当前可用的许可数。      
int availablePermits() 

//获取并返回立即可用的所有许可。    
int drainPermits() 

//返回一个 collection，包含可能等待获取的线程。       
protected  Collection<Thread> getQueuedThreads();

//返回正在等待获取的线程的估计数目。        
int getQueueLength() 

//查询是否有线程正在等待获取。       
boolean hasQueuedThreads() 

//如果此信号量的公平设置为 true，则返回 true。          
boolean isFair() 

//仅在调用时此信号量存在一个可用许可，才从信号量获取许可。          
boolean tryAcquire() 

//如果在给定的等待时间内，此信号量有可用的许可并且当前线程未被中断，则从此信号量获取一个许可。        
boolean tryAcquire(long timeout, TimeUnit unit) 
```

# 例子

```
package threadtest.semaphore;

import java.util.concurrent.Semaphore;

/**
 * @author cxc
 * @date 2018/11/1 13:29
 * 线程类测试信号量
 */
public class SemaphoreThread implements Runnable {
    private Semaphore semaphore;
    private Integer num;

    public SemaphoreThread(Semaphore semaphore, Integer num) {
        this.semaphore = semaphore;
        this.num = num;
    }

    @Override
    public void run() {
        try {
            //获取一个许可
            semaphore.acquire();
            System.out.println("运行中--->" + num);
            Thread.sleep(1000);
            System.out.println("运行结束--->" + num);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //需要释放许可
            semaphore.release();
        }
    }
}

```

```
package threadtest.semaphore;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

/**
 * @author cxc
 * @date 2018/11/1 13:28
 * 信号量
 */
public class SemaphoreTest {
    public static void main(String[] args) {
        //非公平锁  默认是非公平锁
        //Semaphore semaphore = new Semaphore(5,false);
        //公平锁  公平锁
        Semaphore semaphore = new Semaphore(5, true);
        ExecutorService executorService = Executors.newCachedThreadPool();
        int n = 50;
        for (int i = 0; i < n; i++) {
            executorService.execute(new SemaphoreThread(semaphore, i));
        }

    }
}

```
# Semaphore实现互斥锁

在初始化信号量时传入1，使得它在使用时最多只有一个可用的许可，从而可用作一个相互排斥的锁。这通常也称为二进制信号量，因为它只能有两种状态：一个可用的许可或零个可用的许可。按此方式使用时，二进制信号量具有某种属性（与很多 Lock 实现不同），即可以由线程释放“锁”，而不是由所有者（因为信号量没有所有权的概念）。


## 如果信号量只有初始化只有1的时候 这样的话 运行的话最多只有一个可用的许可 其他运行需要释放许可 这样就相当于互斥锁
