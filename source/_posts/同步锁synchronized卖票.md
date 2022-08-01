---
title: 同步锁synchronized卖票
date: 2018-09-17 20:52:19
tags: [多线程]
---
### 运行例子
```
class TicketDemo
{
    public static void main(String args[]) throws Exception
    {
        Ticket t1=new Ticket("第一个窗口");
        t1.start();       /*已经开启的线程。就不需要再开启了*/

        new Ticket("第二个窗口").start();
        new Ticket("第三个窗口").start();
        new Ticket("第四个窗口").start();
    }
}
```
<!--more-->
### 线程类
```
class Ticket extends Thread
{
    private String name;
    Ticket(String name){
        this.name=name;
    }


    private static int tick=100;
    public void run() {
        synchronized (this) {   //变量锁
       // synchronized (Ticket.class) { // 对象锁

            while (true) {
                    if (tick > 0) {
         System.out.println(name + "卖了:--" + tick--);
                    }
                } }
    }

}
```
