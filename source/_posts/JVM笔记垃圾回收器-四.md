---
title: JVM笔记垃圾回收器(四)
date: 2020-07-25 15:06:57
tags: [JVM笔记]
---

# JVM笔记垃圾回收器(四)
## 评估GC的性能指标
```
两个重点:
1.吞吐量
2.暂停时间

设计的时候:
 在最大的吞吐量优先的情况下,降低停顿时间
```
![评估GC的性能指标](/img/2020-07-02/93.png) 

## 垃圾回收器的并发丶并行
<!--more-->
![垃圾回收器的并发丶并行](/img/2020-07-02/87.png)  

![垃圾回收器的并发丶并行](/img/2020-07-02/88.png)


## 何时GC 需要有安全点
![安全点](/img/2020-07-02/89.png)  
![安全点](/img/2020-07-02/90.png)  
![安全点](/img/2020-07-02/91.png)  


## 垃圾回收器

```
1.Serial回收器: 串行回收

2.ParNew回收器: 并行回收

3.Parallel回收器: 吞吐量优先(适合服务端)

4.CMS回收器: 低延迟 (适合响应快的场景)(是并发标记清除)

5.G1回收器: 区域化分布式

6.ZGC回收器: jdk11 可伸缩低延迟的垃圾回收器
```
![垃圾回收器](/img/2020-07-02/94.png)  

### 垃圾回收器与垃圾分代之间的关系
![垃圾回收器与垃圾分代之间的关系](/img/2020-07-02/95.png)   
#### 垃圾回收器之间搭配
![垃圾回收器之间搭配](/img/2020-07-02/96.png)    
![垃圾回收器之间搭配](/img/2020-07-02/97.png)  
## 垃圾回收器介绍

### 如何查看默认的回收器参数
```
-XX:+PrintCommandLineFlags: 查看命令行相关参数(包含使用的垃圾回收器)

使用命令行指令: jinfo -flag 相关垃圾回收器参数 进程ID
能查看是否查看使用当前参数
```

# 各类回收器

### Serial回收器 (串行回收)

```
Serial       采用复制算法
SerialOld    采用标记压缩算法

注意: SerialOld 
 1.与新生代Parallel Scavenge配合使用
 2.作为老年代CMS收集器的后备方案
```
<font color="red">如何设置使用Serial回收器</font>
```
-XX:UseSerialGC 

可以指定年轻代和老年代都使用串行收集器
```

![Serial回收器](/img/2020-07-02/98.png) 
![Serial回收器](/img/2020-07-02/99.png) 



### ParNew回收器  (并行回收)

```
是Serial GC的多线程版本
采用并行回收的方式

注意: 
   只能处理的是新生代
在年轻代也是使用复制算法

注意: ParNew
 除了Serial外,目前只有ParNew GC能与CMS收集器进行配合送

```
<font color="red">如何设置使用ParNew回收器</font>
```
-XX:UseParNewGC  开启GC
-XX:ParallelGCThreads  限制线程数量, 默认开启和CPU数据相同的线程数
```


###  Parallel回收器 并行(吞吐量优先)
### Parallel回收器 并行(吞吐量优先)
```
JDK1.8默认 
新生代 复制算法
老年代 标记压缩算法

注意 :Parallel 也是复制算法 和并行回收
跟ParNew主要区别:
是有一个自适应策略 可以动态分配内存大小等以达到最优的吞吐量
```
<font color="red">如何设置使用Parallel回收器</font>

```
-XX:+UseParallelGC          新生代复制算法 (互相激活 开启一个GC就可以了)
-XX:+UseParallelOldGC       老年代标记压缩算法
-XX:ParallelGCThreads       设置年轻代并行收集器的线程
-XX:MaxGCPauseMillis        设置垃圾收集器最大停顿时间,也就是STW的时间(毫秒) 自适应策略调整大小以控制时间    在该范围内
-XX:GCTimeRatio             垃圾收集时间占总时间的比例 取值范围(0-99) 默认99 也就是垃圾回收的时间不超过1%.也就是=(1/(参数+1));
-XX:+UseAdaptiveSizePolicy  设置收集器是否开启自适应调节策略
```

![自适应调节策略](/img/2020-07-02/100.png) 

### CMS回收器  (并发标记清除  低延迟)

```
在 jdk9废弃 jdk14移除
使用标记清除算法
注意:
    因为垃圾回收的时候 用户线程没有暂停,
所以不能等到堆空间快满的时候进行回收,
会有某个阀值来控制回收的时机.
  
    如果CMS运行期间,预留的内存无法满足程序需要,
将会启动备选方案:Serial回收器 来处理老年代的垃圾
耗时较长
```

<font color="red">如何设置使用CMS回收器</font>
```
-XX:+UseConcMarkSweepGC  开启GC

-XX:CMSlnitiatingOccupanyFraction  
设置堆内存使用率的阀值  ,一旦达到法制就开始回收
jdk5及以前默认是68 也就是老年代68% 开始回收
jdk6以后默认92% 
内存增长快->阀值降低,避免OOM
内存增长慢->阀值提高,降低GC次数

-XX:+UseCMSCompactAtFullCollection
用于指定在执行完FullGC后对内存空间进行压缩整理

-XX:CMSFullGCsBeforeCompaction 
设置在执行多少次Full GC后对内存空间进行压缩整理

-XX:ParallelCMSThreads:
设置CMS的线程数量 默认是:(ParallelGCThreads+3)/4

```
<font color="red">工作原理</font>
```
1.初始标记  
(会非常短暂STW 仅仅只是标记GC Root能直接关联到的对象)

2.并发标记 
(耗时长  
与用户线程一起执行 从GC Root直接关联到的对象遍历整个对象图)

3.重新标记  
(多线程标记 会STW,这个过程比初始标记长且比并发标记时间短,
目的是为了修正并发标记期间,因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录)

4.并发清理 (用户线程一起执行)

```
![CMS回收器工作原理](/img/2020-07-02/101.png)   


###  G1回收器 

```
1.9默认 

可控制延迟的情况下,尽可能获取高的吞吐量
```

<font color="red">如何设置使用G1回收器</font>
```
-XX:+UseG1GC  开启GC

-XX:G1HeapRegionSize 
设置每个Region的大小.
值是2的幂.范围是1MB~32MB之间.即1m 2m 4m 8m 16m 32 m
最多可以划分出2048个区域

-XX:MaxGCPauseMillis 
设置期望达到的最大GC停顿时间
默认值是200ms

-XX:ParallelGCThreads 
设置STW工作线程数的值.最多设置为8

-XX:ConcGCThreads
设置并发标记的线程数.
将n设置为并行垃圾回收线程(ParallelGCThreads)的1/4左右

-XX:InitiatingHeapOccupancyPercent
设置出发并发GC周期的Java堆占用率阀值.
超过此值.就出发GC.默认是45
```

<font color="red">优势</font>
```
1.并行与并发
2.分代收集
3.空间整合  (分区Region:化整为零)
-> 每个region之间是复制算法 整体可看做标记整理
4.可预测的停顿时间 

```
从经验上说:  
小内存应用上CMS的表现大概率会由于G1,  
而G1在大内存应用上则发挥其优势.  
平衡点在6-8GB之间.  

<font color="red">G1何时可替换CMS</font>  
![G1何时可替换CMS](/img/2020-07-02/102.png)

<font color="red">G1回收器工作流程</font>

```
1.年轻代GC
2.老年代并发标记过程
3.混合回收
```
<font color="red">G1回收器详细流程</font>
```

```
![G1年轻代GC](/img/2020-07-02/103.png)  
![G1并发标记过程](/img/2020-07-02/104.png)  
![G1混合回收](/img/2020-07-02/105.png)  
![G1的Full GC](/img/2020-07-02/106.png)  

<font color="red">G1回收器避免全局扫描</font>

```
一个Region不可能是孤立的.一个Region对象可能被其他任意Region中对象引用,
判断对象存活时,是否需要扫描整个java堆才能保证准确性?
这样会导致回收新生代也不得不扫描老年代
降低了Minor GC的效率
```
解决方案:
```
无论G1还是其他分代收集器,JVM都是使用
`Remembered Set`来避免全局扫描

1.每个Region都有一个对应的Remembered Set;

2.每个Reference类型数据写操作时,都会产生一个Write Barrier暂时中断操作;

3.当进行垃圾手机时,在GC根节点的枚举范围加入`Remembered Set`;就可以保证不进行全局扫描,也不会有遗漏;


```

### ZGC回收器 jdk11的

<font color="red">如何设置使用G1回收器</font>
```
-XX:+UnlocakExperimentalVMOptions
-XX:+UseZGC       开启试验参数+开启ZGC

```
![ZGC回收器](/img/2020-07-02/114.png)  



# 7种经典垃圾回收器总结
![7种经典垃圾回收器总结](/img/2020-07-02/107.png)

# 如何选择垃圾回收器
![如何选择垃圾回收器](/img/2020-07-02/108.png)