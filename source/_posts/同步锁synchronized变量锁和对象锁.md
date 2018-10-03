---
title: 同步锁synchronized变量锁和对象锁
date: 2018-09-17 20:54:30
tags: java线程
---

### 例子:
#### 运行main
<!--more-->
```
class ThreadText
{
    public static void main(String args[])
    {
        System.out.println("主线程开始");
        new Text("1").start();    //  这里都是子线程
        new Text("2").start();
        new Text("3").start();
        new Text("4").start();
        new Text("5").start();
        new Text("6").start();
        new Text("7").start();
        new Text("8").start();
        new Text("9").start();
        new Text("10").start();
        new Text("11").start();
        new Text("12").start();
        System.out.println("主线程结束");

    }
}.     
```
---

#### 线程类
```
class Text extends Thread   //具体实现方式
{
    private String name;
    public Text(String name)
    {
       this.name=name;
    }
    public void run()
    {
        synchronized (Text.class){           
        //这里用对象来当做锁 因为是不同对象同时
            System.out.println(name+"开始");
            try {
                Thread.sleep(1000);       
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(name+"结束");
    }
}

}
```