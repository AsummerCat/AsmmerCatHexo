---
title: 面试相关-JVM
date: 2020-02-03 22:45:39
tags: [面试相关,jvm]
---

## JVM

### 内存模型

```
线程独占   ->本地方法栈  栈 程序计数器
线程共享   ->堆  方法区 1.8使用元空间(Metaspace) 


栈:  又称方法栈,线程私有的,线程执行方法是都会创建一个栈帧,用来存储局部变量表,操作栈,动态链接,方法出口等信息.调用方法时执行入栈,方法返回式执行出栈.

本地方法栈: 与栈类似,也是用来保存执行方法的信息.执行Java方法是使用栈,执行Native方法时使用本地方法栈.

程序计数器: 保存着当前线程执行的字节码位置,每个线程工作时都有独立的计数器,只为执行Java方法服务,执行Native方法时,程序计数器为空.

堆:JVM内存管理最大的一块,对被线程共享,目的是存放对象的实例,几乎所欲的对象实例都会放在这里,当堆没有可用空间时,会抛出OOM异常.根据对象的存活周期不同,JVM把对象进行分代管理,由垃圾回收器进行垃圾的回收管理

方法区:   它用于存储已被虚拟机加载的类信息, 常量, 静态变量, 即时编译器编译后的代码等数据。
```

 <!--more-->

### volatile

-```
  - 可见性保证：强制变量的赋值会刷新回主内存，强制变量的读取会从主内存中重新加载，保证不同的线程总是能看到该变量的最新值。
  - 有序性保证：通过指令重排序保证变量读写的有序性
 ```

### JVM加载顺序

![JVM加载顺序](/img/2020-02-03/5.png)

```
深绿色表示加载过程，浅绿色表示生命周期。

加载：通过类的完全限定名找到字节码文件，通过字节码文件创建class对象。

验证：图中始终验证方法。

准备：为static修饰的变量分配内存，初始值0或者null。（final不分配，因为在编译时已经分配）

解析：图中。

初始化：看图中解释，若类的父类没有初始化，则先初始化父类的静态块和静态变量，只有对类的主动使用时才会初始化。

初始化的出发条件：创建类实例、访问类静态变量或者静态方法、class.forName()发射加载、某个子类被初始化。
使用：实例化。

卸载：java前三种类加载器的加载的类不会被卸载，用户自定义类加载器加载的类才会被卸载。
```

### 类加载器与加载模式

![JVM加载顺序](/img/2020-02-03/7.png)

```
类加载器：启动类加载器、扩展类加载器、应用/系统加载器、用户自定义加载器。

双亲委派模式好处：

避免类的重复加载；
防止java系统类被篡改。
```

# 调优参数

### JVM的参数类型

```
1. 标配参数   (稳定参数)    例如:查看版本号 -version , -help  , java -showversion

2. x参数(了解)     例如: -Xint 解释执行  -Xcomp JIT编译执行  -Xmixed 混合执行  

3. xx参数 (重点使用)
    | boolean类型  : -XX:+或者-属性值   表示开启或关闭某个属性 例如:开启GC日志 使用某个GC回收策略
    | KV设值类型    : -XX:属性key=属性value               例如:-XX:MetaspaceSize=128m 
    
```

### 查看JVM参数初始值和修改

```
1. 使用 jps 查看进程后 再用jinfo -flags 进程号 查看运行的进程JVM参数

2. 使用 java -XX:+PrintFlagsInitial    查看初始化默认值 (推荐)

3. 使用 java -XX:+PrintFlagsFinal -version           查看修改和更新的内容
        |java -XX:+PrintFlagsFinal -XX:MetaspaceSize=512m 运行类名  ->这是运行时修改

4. 使用 java -XX:+PrintCommandLineFlags -version 查看初始值 (主要容易看垃圾回收器)
```

### 常用JVM参数

```
1. -Xms            设置初始化内存

2. -Xmx            设置最大内存

3. -Xss            设置线程 栈内存大小 栈深度

4. -Xmn            设置新生代大小

5. -XX:MetaspaceSize   设置元空间大小

6. -XX:PrintGCDetails    显示GC明细日志

7. -XX:SurvivorRatio    设置幸存区比例占比 =8        默认8:1:1     新生代默认交换15次晋升老年代

8. -XX:NewRatio         设置老年代的占比大小  =2    默认 新生代1老年代2 

9. -XX:MaxTenuringThreshold  设置垃圾最大年龄   默认 15次   0-15之间
```



# 引用 Reference

### 强引用95%   (打死不被GC)

```
强引用 就算出现OOM也不回收对象 无法被垃圾回收
强引用指向一个对象
意思是: 一个对象赋给一个引用变量

造成java内存泄漏的主要原因之一

代码:
  o1 =new o1();
  o2=o1
  GC
  o2仍存在
```

### 软引用 (内存不够的情况下 会被GC)

```
SoftReferrence类实现
实现:
   SoftReferrence<Object> o1=new SoftReferrence<Object>();

例如高效缓存  对内存敏感的程序

内存充足不回收,内存不足会被回收  
```

### 弱引用(只要GC就回收)

```
只要有GC一律回收

WeakReferrence类实现

例如:
  1.读取大批量本地文件加入缓存 加快查看 软引用弱引用都可以
  2. mybatis里面缓存部分 大批量的使用软引用和弱引用
```

#### 弱引用->WeakHashMap(缓存常用)

```
弱HashMap
WeakHashMap:   
     跟普通的hashMap的区别: key的引用被修改为null 被GC后   键值对会被回收 

```

### 虚引用

```
PhantomReference类实现

形同虚设 不决定对象的生命周期 

必须和引用队列(ReferenceQueue)联合使用
回收前 需要放入队列中保存一下

主要作用:  跟踪对象被垃圾回收的状态
只能在GC对象呗finalize以后 回收之后的做进一步处理 

类似Aop的后置通知
```



# OOM的理解

```
java.lang.StackOverflowError  栈内存溢出

java.lang.OutOfMemoryError: java heap space  堆内存溢出 

java.lang.OutOfMemoryError: GC overhead limit exceeded 
GC时间过长并且回收效率低 98%的时间回收不到2%的堆内存 

java.lang.OutOfMemoryError: Direct buffer memory    
直接内存不足  电脑内存不足  
造成原因: netty会出现allocate()导致直接调用堆外内存  
解决方案: 可修改-XX:maxDirectMemory设置直接内存大小

java.lang.OutOfMemoryError: unable to create new native thread 
不能再创建本地线程
造成原因:  1.一个进程创建太多线程,超出系统承载上限   
          2.服务器不允许创建这么多线程 linux默认1024线程
解决方案:  1.linux修改句柄 扩大创建线程数量
              | 查看当前用户最大可执行线程数   ulimit -u
              | 查看系统配置   vim /etc/security/limits.d/90-nproc.conf
              | 添加张三用户的 线程数
          2.降低引用的线程数


java.lang.OutOfMemoryError: Metaspace   元空间溢出
设置元空间大小 :-XX:Metaspacesize
              -XX:MaxMetaspacesize

元空间存放信息:
      1.虚拟机加载的类信息
      2.常量池
      3.静态变量
      4.即时编译后的代码


```





# 垃圾回收部分

### GC区域分块

```
分代回收
   新生代 -> minorGC
   老年代  -> FUllGC
```

### 回收算法4种

```
引用记数算法 (不推荐)
复制算法 ->新生代
标记清除算法 ->老年代
标记整理算法 ->老年代
标记压缩算法 ->老年代
```

### 5大垃圾回收器

```
Serial     串行化回收           单线程
Parallal   并行回收             多线程      适合弱交互 比如科学计算平台 
CMS        并发标记清除回收      (互联网常用)用户线程和垃圾回收同时进行  对响应时间有要求
G1         (1.9默认)分块回收      适用于堆内存很大的情况下 并发回收
ZGC        (java11)低延迟 停段时间不超过10ms
```

### 具体实现JVM参数(6大垃圾收集器 )

```
1.-XX: +UseSerialGC          串行收集器                   默认情况下:  Serial+SerialOld

2. -XX: +UseParallelGC      并行收集器 1.8默认吞吐量优先   默认情况下: ParallelScavengeGC+ParallelOld
   上下两个互相激活 开启一个就好了
   -XX: +UseParallelOldGC     老年代并行收集器 
   -XX:ParallelGCThreads=数字N 表示开启多少个GC线程  CPU>8 N=5/8   CPU<8 N=实际  


3.-XX: +UseConcMarkSweepGC   CMS收集器     最短停顿时间     推荐使用: ParNew+CMS+ SerialOld(后备) 
    因为是CMS 并发收集->标记清除 并没有压缩操作  CMS无法处理垃圾的时候启动备用单线程收集器进行阻塞回收
     可以添加参数-> -XX: CMSFullGCsBeForeCompaction 默认0 (指定多次GC后 进行一次压缩的FULL GC)

4.-XX: +UseG1GC              G1收集器                     默认情况下: 标记整理
    每块1m-32m不等   -XX:G1HeapRegionSize=n 可指定分区大小 必须是2的幂
    默认分为1024个区域,最大可分为2048个分区 ,也就是最大支持64G内存
    添加了预测时间 用户可以指定期望的停顿时间
    标记整理 不产生碎片
    


5.-XX: +UseParNewGC          新生代并行收集器              默认情况下:   ParNew+SerialOld
 可搭配老年代的CMS收集器
       
6. SerialOld 老年代串行回收 (不用书写)

```

### G1的参数

```
1. -XX:UseG1GC     使用GC

2. -XX:G1HeapRegionSize=n    设置G1区域大小范围1M-32M

3. -XX:MaxGCPauseMillis=n    设置GC最大停顿时间(毫秒) 不保证肯定

4. -XX:InitiatingHeapOccupancyPenrcent=n 堆占用多少时候触发GC ,默认 45

5. -XX:ConcGCThreads=n       并发GC使用的线程数

6. -XX:G1ReservePercent=n     设置作为空闲空间的预留内存百分比 降低目标空间移除的风险 默认 10%
```

### GC知识点整理

```
新生代回收器：SerialGC ParNewGc ParallelScavengeGC
名称	                  串行/并行/并发	回收算法	适用场景	         可以与cms配合
SerialGC	                   串行	      复制	   单cpu   	          否
ParNewGC	                   并行	      复制	   多cpu    	          是
ParallelScavengeGC	         并行	      复制	   多cpu且关注吞吐量	    否

三种老生代回收器
名称	                  串行/并行/并发	回收算法	适用场景
SerialOldGC	                串行	   标记整理	  单cpu
ParNewOldGC	                并行	   标记整理	  多cpu
CMS	并发，几乎不会暂停用户线程	标记清除	多cpu且与用户线程共存
CMS收集器的优点：并发收集、低停顿

G1收集器 分块收集 1.9默认的垃圾回收机制
G1适合对最大延迟有要求的场合，ZGC适合64位对大内存有要求的场合

还有一个ZGC收集器 11提供 是一个可伸缩的、低延迟的垃圾收集器
停顿时间不会超过10ms
停顿时间不会随着堆的增大而增大（不管多大的堆都能保持在10ms以下）
可支持几百M，甚至几T的堆大小（最大支持4T）

```



# 真题

##  **JDK8中永久代向元空间的转换原因**

```
1、字符串存在永久代中，容易出现性能问题和内存溢出。

2、类及方法的信息等比较难确定其大小（比如动态加载类时），因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。
```

## 编译期会对指令做什么操作?

```
指令重排 优化执行效率
如A 和 B 无关 可以调整执行顺序
如果 C 必须执行在AB之后 则无法重排
```

## 什么情况下会造成fullGC

```
新生代晋升
老年代空间不足
永久代空间不足
```

## 简单描述下volatile能解决什么问题?

```
强制主内存读写同步，防止指令重排序
根据volatile的可见性保证 和有序性保证  读取volatile的内容直接读取到内存里的
```

## 对象引用有几种?

```
强引用：不会被GC回收
弱引用：每次GC都会被回收
虚引用：必须和引用兑现联合使用，跟总一个对象被垃圾回收的过程
软引用：空间不足会被GC回收
```

## 使用过哪些JVM调试工具

```
JPS -l:       查看java进程

jstack 进程号:       线程分析工具(堆栈异常分析)

jinfo -flag printGCDetails 进程号:     获取进程的详细信息 是否开启参数
                或者 -flags 进程号:   显示所有参数
                
jmap -heap 进程ID    获取映射堆快照

jvisualvm          可视化工具

jconsole           可视化工具

```

## JVM执行模式有几种?

```
解释执行   不经过jit直接由解释器解释执行所有字节码  ->字节码翻译成本地码再执行
编译执行   判断是否 "热点代码" 是的话直接走JIT编译执行 机器码
混合执行   新版本的jvm默认都是采用混合执行模式
```

## GC算法的实现和适用场景

```
CMS: 以获取最短回收停顿时间为目标的收集器
G1:  G1适合对最大延迟有要求的场合  可预测停顿
ZGC: ZGC适合64位对大内存有要求的场合
```

## 类加载过程

```
加载: 文件加载到JVM
验证: 验证 字节码,文件格式,元数据,符号引用
准备: 准备类变量的内存
解析: 解析 引用替换,字段解析,接口解析,方法解析
初始化: 静态块,静态变量
使用: 实例化
卸载: GC

Java自带的加载器加载的类,在虚拟机的生命周期中是不会被卸载的,只有用户自定义的加载器加载的类才可以被卸.
```

## JVM加载类的机制

```
双亲委派机制
          即加载器加载类时先把请求委托给自己的父类加载器执行,直到顶层的启动类加载器.父类加载器能够完成加载则成功返回,不能则子类加载器才自己尝试加载.
   优点:
         1.避免类的重复加载
         2.避免Java的核心API被篡改
```

## 什么是垃圾&什么是GC Roots?

```
垃圾: 
    1. 引用不可达  枚举根节点做可达性分析
    2. 引用计数法
     
GC Roots:
  链路追踪
  跟踪GC的根节点  从(GC root对象)根节点出发 ->能被遍历到的对象就判断为存活 ,没有遍历到的判断为死亡
  
  那些对象能成为GC root的对象 :
          定一个集合的引用作为根节点出发
          集合为:
          1.虚拟机栈(栈帧中的局部变量表)中引用的对象         new 对象
          2.方法区中的类静态属性引用的对象                  static 
          3.方法区中常量引用的对象                         final
          4.本地方法栈native引用的对象                     unsafe
```

## GC收集方式 G1和CMS的区别

```
CMS 可能产生内存碎片                     算法:标记清除
G1   尽可能不产生碎片  可控制预测停顿时间    算法:复制算法 +标记整理
```

## 生产服务器变慢了 ,诊断思路和性能评估?

```
1. 查看系统负载能力 
   | top命令查看   CPU和内存 及其右上角的load average(负载均衡 1m 10m 15m) 3个数值/3*100 >60% 负载重
   |    执行命令后 按键盘 1 查看具体哪个CPU负载高
   | 或者执行 uptime 直接查看负载均衡 也是3合1大于60%
   
   
2.查看CPU  
  | vmstat -n 2 3 查看间隔2秒 3次
  | 具体查看 -procs   r:运行和等待CPU时间片的进程数 最大不超过MaxCpuNum 2倍  b:等待资源数 如磁盘io和网络io
             -cpu    us: 用户进程消耗的CPU百分比 大于50%优化程序  sy :内核进程消耗的CPU时间比
  | 或者  mpstat -P ALL 2 命令  查看所有CPU
             
             
 3.查看指定进程使用信息
    pidstat -u 1(间隔) -p 进程编号       查看指定进程使用CPU的用量分解信息
    pidstat -p 进程号 -r  2              查看内存情况
    pidstat -d 1(间隔) -p 进程号          查看磁盘读写情况
    
    
 
 4.内存查看
   free -m 命令 
   
   
 5. 查看磁盘剩余空间
    df -h 命令 查看剩余空间 (-h表示换算为G)


6.查看磁盘IO   (重点排查对象)
    iostat -xdk 2 3 命令 查看
        |rkB/s 每秒读取数据量kb
        |wkB/s 每秒写入数据量kb
        |svctm I/O请求的平均服务时间,单位毫秒
        |await I/O请求的平均等待时间.单位毫秒 :值越小,性能越好
        |%util 一秒中有多少百分比用于I/O操作 ,接近100% 表示磁盘带宽跑满 需要优化程序或者增加磁盘
        |svctm和await的值很接近表示没有I/O等待,磁盘性能好
        |如果await的值远高于svctm的值,则表示I/O队列等待太长,表示需要优化程序或者更换更快的磁盘
        
        
        
 7. 网络IO
    ifstat -1命令 (没有该命令就下载)  查看各网卡信息
    
 
```

## 常用的linux命令

```
top         获取整机情况
vmstat      cpu情况
free        查看内存
df          查看磁盘
iostst      磁盘io
ifstat      网络io
uptime      查看系统负载
```

## CPU占用过高 如何定位?(重点)

```
1.根据 TOP命令 找到CPU占用过高的程序
2. 使用ps ef|grep 或者JPS -l 找到后台程序  
3.定位到具体线程或者代码   命令:  ps -mp 进程 -o -THREAD,tid,time     (tid:线程id)
4.将需要的线程ID转换为16进制格式(小写)   命令 print "%x\n" 有问题的线程ID
5.具体检测线程: jstack 进程ID | grep 16进制tid -A60  (获取前60行)
```

