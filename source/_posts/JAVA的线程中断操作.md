---
title: JAVA的线程中断操作
date: 2021-10-09 00:00:12
tags: [java,多线程]
---

# java的线程中断
stop是强制中断线程
interrupt 是标记中断位置 等待任务结束

## 相关代码

### 用来判断当前线程是否中断
```
Thread.currentThread().isInterrupted();

后面加一个Thread.sleep(5000);
会抛出异常
java.lang.InterruptedException: sleep interrupted

```

<!--more-->

### 用来中断当前线程(内部方法)

```
在线程体内
1.标记中断位置
this.interrupt();
2.判断线程是否中断 中断后退出
if (Thread.currentThread().isInterrupted()){
				System.out.println("exit MyThread");
				break;
			}

```


### 线程外部判断线程是否中断(外部方法)
```

public class MyThread1 extends Thread {
	public MyThread1() {

	}

	@Override
	public void run() {
		try {
			work2();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

		public void work2() throws InterruptedException {
			while (true) {
				System.out.println("111");
				if (Thread.currentThread().isInterrupted()) {
					System.out.println("222");
					System.out.println("333");
					System.out.println("C isInterrupted()=" + Thread.currentThread().isInterrupted());
					Thread.sleep(5000);
					//System.out.println("D isInterrupted()=" + Thread.currentThread().isInterrupted());
				}
			}
		}



	public static void main(String[] args) throws InterruptedException {
		MyThread1 thread = new MyThread1();
		thread.start();
		System.out.println(thread.getState());
		System.out.println("开始中断线程");
		thread.interrupt();
		Thread.sleep(1000);//等到thread线程被中断之后
		System.out.println(thread.isInterrupted());

		System.out.println(thread.getState());

	}
}


```


## 线程内部中断 完整方法
```

public class MyThread extends Thread {
	public MyThread() {

	}

	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(Thread.currentThread() + ":" + i);
			if (i == 5) {
            this.interrupt();
			}
			if (Thread.currentThread().isInterrupted()){
				System.out.println("exit MyThread");
				break;
			}
		}
	}

	public static void main(String[] args) {
		MyThread mThread1 = new MyThread();
		mThread1.start();
	}
}


```

## 线程外部中断 完整代码
```


```
