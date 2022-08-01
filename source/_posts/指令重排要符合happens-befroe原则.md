---
title: 指令重排要符合happens-befroe原则
date: 2020-05-13 10:25:41
tags: [jvm,java,volatie]
---

# 指令重排要符合happens-befroe原则

```
1、程序次序法则，如果A一定在B之前发生，则happen before；
2、监视器法则,对一个监视器的解锁一定发生在后续对同一监视器加锁之前；
3、Volatie变量法则：写volatile变量一定发生在后续对它的读之前；
4、线程启动法则：Thread.start一定发生在线程中的动作；
5、线程终结法则：线程中的任何动作一定发生在括号中的动作之前（其他线程检测到这个线程已经终止，从Thread.join调用成功返回，Thread.isAlive()返回false）；
6、中断法则：一个线程调用另一个线程的interrupt一定发生在另一线程发现中断；
7、终结法则：一个对象的构造函数结束一定发生在对象的finalizer之前；
8、传递性：A发生在B之前，B发生在C之前，A一定发生在C之前。
```

