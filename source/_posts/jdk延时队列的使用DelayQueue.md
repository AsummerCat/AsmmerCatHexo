---
title: jdk延时队列的使用DelayQueue
date: 2019-11-12 11:30:16
tags: [jdk,java, DelayQueue,Queue]
---

# Java延时队列DelayQueue的使用

# 核心方法

```
		DelayQueue<DelayedTask> delayQueue = new DelayQueue<DelayedTask>();

DelayedTask=> 实现任务接口 implements Delayed 
delayQueue.offer(delayedTask); 添加任务 如果队列已满，则返回false
 delayQueue.take();  消费任务    移除并返回队列头部的元素       如果队列为空，则阻塞

```

# 延时队列测试demo

## Main函数

```
public class DelayQueueTest {
	public static void main(String[] args) {
		DelayQueue<DelayedTask> delayQueue = new DelayQueue<DelayedTask>();
		for (int i = 0; i < 100; i++) {
			delayQueue.add(new DelayedTask(1000 * i, "测试数据" + i));
		}
		//开启线程100毫秒生成一个任务
		producer(delayQueue);
        //开启线程消费超时任务
		consumer(delayQueue);
	}

```

<!--more-->

## 延时队列的内容对象

```java
/**
	 * 延时队列的内容对象
	 */
	static class DelayedTask implements Delayed {
		private final long delay; //延迟时间
		private final long expire;  //到期时间
		private final String msg;   //数据
		private final long now; //创建时间

		public DelayedTask(long delay, String msg) {
			this.delay = delay;
			this.msg = msg;
			expire = System.currentTimeMillis() + delay;    //到期时间 = 当前时间+延迟时间
			now = System.currentTimeMillis();
		}

		/**
		 * 需要实现的接口，获得延迟时间   用过期时间-当前时间
		 *
		 * @param unit
		 * @return
		 */
		@Override
		public long getDelay(TimeUnit unit) {
			//根据过期时间-当前时间
			return unit.convert(this.expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
		}

		/**
		 * 用于延迟队列内部比较排序   当前时间的延迟时间 - 比较对象的延迟时间
		 *
		 * @param o
		 * @return
		 */
		@Override
		public int compareTo(Delayed o) {
			return (int) (this.getDelay(TimeUnit.MILLISECONDS) - o.getDelay(TimeUnit.MILLISECONDS));
		}

		@Override
		public String toString() {
			final StringBuilder sb = new StringBuilder("DelayedTask{");
			sb.append("delay=").append(delay);
			sb.append(", expire=").append(expire);
			sb.append(", msg='").append(msg).append('\'');
			sb.append(", now=").append(now);
			sb.append('}');
			return sb.toString();
		}
	}
```

## 每100毫秒创建一个对象，放入延迟队列，延迟时间1毫秒

```
	/**
	 * 每100毫秒创建一个对象，放入延迟队列，延迟时间1毫秒
	 *
	 * @param delayQueue
	 */
	private static void producer(final DelayQueue<DelayedTask> delayQueue) {
		new Thread(() -> {
			while (true) {
				try {
					TimeUnit.MILLISECONDS.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}

				DelayedTask element = new DelayedTask(1000, "test");
				delayQueue.offer(element);
				System.out.println("添加" + element.toString());
			}

		}).start();
	}
```

## 消费任务

```java
	/**
	 * 消费任务
	 *
	 * @param delayQueue
	 */
	private static void consumer(final DelayQueue<DelayedTask> delayQueue) {
		new Thread(() -> {
			while (true) {
				DelayedTask element = null;
				try {
					element = delayQueue.take();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				System.out.println(System.currentTimeMillis() + "---" + element);
			}
		}).start();
	}
```



