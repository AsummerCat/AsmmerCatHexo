---
title: queue的使用
date: 2018-10-30 17:16:43
tags: [java]
---

# 队列的使用

## 基本概念

* 先进先出~~
<!--more-->

* Java提供的线程安全的Queue可以分为阻塞队列和非阻塞队列
* 其中阻塞队列的典型例子是BlockingQueue 
* 非阻塞队列的典型例子是ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。 

* 其他的还有 :AbstractQueue, ArrayBlockingQueue, ConcurrentLinkedQueue, LinkedBlockingQueue, DelayQueue, LinkedList, PriorityBlockingQueue, PriorityQueue和ArrayDqueue。

```
 BlockingQueue是个接口，有如下实现类：
       1. ArrayBlockQueue：一个由数组支持的有界阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。创建其对象必须明确大小，像数组一样。

       
       2. LinkedBlockQueue：一个可改变大小的阻塞队列。此队列按 FIFO（先进先出）原则对元素进行排序。创建其对象如果没有明确大小，默认值是Integer.MAX_VALUE。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。 


       3. PriorityBlockingQueue：类似于LinkedBlockingQueue，但其所含对象的排序不是FIFO，而是依据对象的自然排序顺序或者是构造函数所带的Comparator决定的顺序。


       4. SynchronousQueue：同步队列。同步队列没有任何容量，每个插入必须等待另一个线程移除，反之亦然。
```

## 基本方法使用

对于Queue来说，就是一个FIFO（先进先出）的队列，添加元素只能在队尾，移除只能在队首。

对于这一组方法，成功返回true，在操作失败时抛出异常，这是与下面一组方法的主要区别。

```
add()        增加一个元索                     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
remove()     移除并返回队列头部的元素           如果队列为空，则抛出一个NoSuchElementException异常
element()    返回队列头部的元素                如果队列为空，则抛出一个NoSuchElementException异常

```

 
这一组，成功返回true，失败时返回一个特殊值(取决于操作，为NULL或false)，offer(E e)操作是专为容量受限的队列实现而设计的；在大多数实现中，插入操作不会失败。   

```
offer()        添加一个元素并返回true          如果队列已满，则返回false
poll()         移除并返问队列头部的元素         如果队列为空，则返回null
peek()         返回队列头部的元素              如果队列为空，则返回null
put()          添加一个元素                   如果队列满，则阻塞
take()         移除并返回队列头部的元素         如果队列为空，则阻塞

```
remove、element、offer 、poll、peek 其实是属于Queue接口。 

# 简单使用例子

```
package listanalyze;

import java.util.LinkedList;
import java.util.Queue;

/**
 * @author cxc
 * @date 2018/10/30 17:29
 * 队列的简单使用
 * 需要注意事项: 队列没有值的话 element()丶remove()会报错,而 poll()丶peek()不会报错
 * Exception in thread "main" java.util.NoSuchElementException;
 */
public class QueueTest {

    public static void main(String[] args) {
        Queue<Integer> queue = new LinkedList<>();
        //入队
        //添加 add()
        queue.poll();
        queue.add(1);
        queue.add(2);
        boolean add = queue.add(3);
        queue.forEach(data -> System.out.println("初次返回结果:--->" + data));

        //添加成功 有返回值 offer()
        boolean offer = queue.offer(4);
        boolean offer1 = queue.offer(5);
        boolean offer2 = queue.offer(6);
        queue.forEach(data -> System.out.println("再次返回结果:--->" + data));

        System.out.println("------------------------");
        //获取第一个元素并删除
        //方法 poll()    让第一个元素出队
        System.out.println("poll=" + queue.poll());
        queue.forEach(data -> {
            System.out.println("初次poll后剩下:-->" + data);
        });
        System.out.println("------------------------");
        //获取第一个元素并删除
        //方法 remove()    让第一个元素出队
        System.out.println("remove=" + queue.remove());
        queue.forEach(data -> {
            System.out.println("初次remove后剩下:-->" + data);
        });
        System.out.println("------------------------");
        //查看第一个元素 不出队
        //方法 peek()
        System.out.println("peek=" + queue.peek());
        queue.forEach(data -> {
            System.out.println("初次peek后剩下:-->" + data);
        });
        System.out.println("------------------------");
        //查看第一个元素 不出队
        //方法 element()
        System.out.println("element=" + queue.element());
        queue.forEach(data -> {
            System.out.println("初次element后剩下:-->" + data);
        });
        System.out.println("------------------------");
    }


}

```

* 结果:

```
初次返回结果:--->1
初次返回结果:--->2
初次返回结果:--->3
再次返回结果:--->1
再次返回结果:--->2
再次返回结果:--->3
再次返回结果:--->4
再次返回结果:--->5
再次返回结果:--->6
------------------------
poll=1
初次poll后剩下:-->2
初次poll后剩下:-->3
初次poll后剩下:-->4
初次poll后剩下:-->5
初次poll后剩下:-->6
------------------------
remove=2
初次remove后剩下:-->3
初次remove后剩下:-->4
初次remove后剩下:-->5
初次remove后剩下:-->6
------------------------
peek=3
初次peek后剩下:-->3
初次peek后剩下:-->4
初次peek后剩下:-->5
初次peek后剩下:-->6
------------------------
element=3
初次element后剩下:-->3
初次element后剩下:-->4
初次element后剩下:-->5
初次element后剩下:-->6
------------------------

```

# 注意事项

## 报错情况
```
add(),element(),remove()会报错,  
而 offer(),poll(),peek()不会报错

Exception in thread "main" java.util.NoSuchElementException;

```