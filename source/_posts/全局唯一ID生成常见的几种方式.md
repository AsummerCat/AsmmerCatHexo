---
title: 全局唯一ID生成常见的几种方式
date: 2019-05-15 23:47:22
tags: [分布式ID,雪花算法,java]
---

# 全局唯一ID生成常见的几种方式

全局唯一ID生成常见的几种方式：

*  1（twitter/snowflake）雪花算法

* 2 利用数据库的auto_increment特性  

* 3 UUID

* 4 其他（如redis也有incr，redis加lua脚本实现twitter/snowflake算法）



<!--more-->

![介绍](/img/2019-5-15/title.png)

# 数据库自增

可以设置数据库自增 这个最简单   

优点: 不用设置其他东西直接使用

缺点: 如果在分布式环境会出现性能问题 多数据库的话 会涉及到id重复的情况 

优化: 设置每个库设置不同的步长 但是4个库就会冲突了

```java
例如:
A  0 3 6 9
B  1 4 7 10
C  2 5 8 11
D  3 6 9 12
这里6就冲突了

还会出现一个问题就是   如果一个挂了 要是id产生错误 会产生蝴蝶效应 后面的全部都不行
```



# UUID

常见的方式。可以利用数据库也可以利用程序生成，一般来说全球唯一

```java
public static void main(String[] args) throws Exception {
    System.out.println(UUID.randomUUID());
}
```



（1）当前日期和时间，UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒又生成一个UUID，则第一个部分不同，其余相同。

（2）时钟序列。

（3）全局唯一的IEEE机器识别号，如果有网卡，从网卡MAC地址获得，没有网卡以其他方式获得。

优点：

1）简单，代码方便。

2）生成ID性能非常好，基本不会有性能问题。

3）全球唯一，在遇见数据迁移，系统数据合并，或者数据库变更等情况下，可以从容应对。

缺点：

1）没有排序，无法保证趋势递增。

2）UUID往往是使用字符串存储，查询的效率比较低。

3）存储空间比较大，如果是海量数据库，就需要考虑存储量的问题。

4）传输数据量大

5）不可读。

变种的UUID

1）为了解决UUID不可读，可以使用UUID to Int64的方法。

2）为了解决UUID无序的问题，NHibernate在其主键生成方式中提供了Comb算法（combined guid/timestamp）。保留GUID的10个字节，用另6个字节表示GUID生成的时间（DateTime）。

# 雪花算法snowflake

使用了long类型，long类型为8字节工64位。可表示的最大值位2^64-1（18446744073709551615，装换成十进制共20位的长度，这个是无符号的长整型的最大值）。

单常见使用的是long 不是usign long所以最大值为2^63-1（9223372036854775807，装换成十进制共19的长度，这个是long的长整型的最大值）

![雪花算法](/img/2019-5-15/雪花算法.png)

## 源码

```java


public class IdGen {
    private long workerId;
    private long datacenterId;
    private long sequence = 0L;
    private long twepoch = 1288834974657L;
     //Thu, 04 Nov 2010 01:42:54 GMT
    private long workerIdBits = 5L;
     //节点ID长度
    private long datacenterIdBits = 5L;
     //数据中心ID长度
    private long maxWorkerId = -1L ^ (-1L << workerIdBits);
     //最大支持机器节点数0~31，一共32个
    private long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
     //最大支持数据中心节点数0~31，一共32个
    private long sequenceBits = 12L;
     //序列号12位
    private long workerIdShift = sequenceBits;
     //机器节点左移12位
    private long datacenterIdShift = sequenceBits + workerIdBits;
     //数据中心节点左移17位
    private long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
     //时间毫秒数左移22位
    private long sequenceMask = -1L ^ (-1L << sequenceBits);
     //最大为4095
    private long lastTimestamp = -1L;
    
    private static class IdGenHolder {
        private static final IdGen instance = new IdGen();
    }
    
    public static IdGen get(){
        return IdGenHolder.instance;
    }

    public IdGen() {
        this(0L, 0L);
    }

    public IdGen(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }
    
    //具体内容
    public synchronized long nextId() {
        long timestamp = timeGen();
        //获取当前毫秒数
        //如果服务器时间有问题(时钟后退) 报错。
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(String.format(
                    "Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }
        //如果上次生成时间和当前时间相同,在同一毫秒内
        if (lastTimestamp == timestamp) {
            //sequence自增，因为sequence只有12bit，所以和sequenceMask相与一下，去掉高位
            sequence = (sequence + 1) & sequenceMask;
            //判断是否溢出,也就是每毫秒内超过4095，当为4096时，与sequenceMask相与，sequence就等于0
            if (sequence == 0) {
                timestamp = tilNextMillis(lastTimestamp);
                 //自旋等待到下一毫秒
            }
        } else {
            sequence = 0L;
             //如果和上次生成时间不同,重置sequence，就是下一毫秒开始，sequence计数重新从0开始累加
        }
        lastTimestamp = timestamp;
        
         long longStr= ((timestamp - twepoch) << timestampLeftShift) | (datacenterId << datacenterIdShift) | (workerId << workerIdShift) | sequence;
         // System.out.println(longStr);
         return longStr;
    }

    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    protected long timeGen() {
        return System.currentTimeMillis();
    }

}
```

测试用的电脑CPU为I5-4210U，内存8G，JDK为1.7.0_79，系统是64位WIN 7，使用-server模式。  

## 测试代码

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

import org.junit.Test;

public class GeneratorTest {

    @Test
    public void testIdGenerator() {
        long avg = 0;
        for (int k = 0; k < 10; k++) {
            List<Callable<Long>> partitions = new ArrayList<Callable<Long>>();
            final IdGen idGen = IdGen.get();
            for (int i = 0; i < 1400000; i++) {
                partitions.add(new Callable<Long>() {
                    @Override
                    public Long call() throws Exception {
                        return idGen.nextId();
                    }
                });
            }
            ExecutorService executorPool = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
            try {
                long s = System.currentTimeMillis();
                executorPool.invokeAll(partitions, 10000, TimeUnit.SECONDS);
                long s_avg = System.currentTimeMillis() - s;
                avg += s_avg;
                System.out.println("完成时间需要: " + s_avg / 1.0e3 + "秒");
                executorPool.shutdown();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        System.out.println("平均完成时间需要: " + avg / 10 / 1.0e3 + "秒");
    }
}
```



每次1.038秒生成140万个ID，除了第1次时间在3秒左右和第2次1.6秒左右，其余8次都在0.7秒左右。如果使用更好的硬件，测试数据肯定会更好。因此从大的方向上看，单节点的ID生成器基本上可以满足我们的需要了。

* tips: 需要注意的是，该值只是一个唯一值，但并不能保证会是一个顺序值，就是说两个ID之间可能会跳一些数字，所以对于一些有特殊需求的业务来说请注意这个差异。



# redis 

redis的incr 和INCRBY来实现可以实现自增

思路 incr的话没有就可以创建 当前key 从0开始  然后自增 



# 其他

* redis-lua脚本实现twitter/snowflake算法。

* MongoDB的ObjectId。