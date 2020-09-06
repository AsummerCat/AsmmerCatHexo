---
title: 排查JAVA问题常用工具单
date: 2019-09-06 22:46:04
tags: [java,虚拟机调优]
---

# linux命令

## tail
```
tail -300f shopbase.log #倒数300行并进入实时监听文件写入模式
```

## grep
```
grep forest f.txt     #文件查找
grep forest f.txt cpf.txt #多文件查找
grep 'log' /home/admin -r -n #目录下查找所有符合关键字的文件
cat f.txt | grep -i shopbase    
grep 'shopbase' /home/admin -r -n --include *.{vm,java} #指定文件后缀
grep 'shopbase' /home/admin -r -n --exclude *.{vm,java} #反匹配
seq 10 | grep 5 -A 3    #上匹配
seq 10 | grep 5 -B 3    #下匹配
seq 10 | grep 5 -C 3    #上下匹配，平时用这个就妥了
cat f.txt | grep -c 'SHOPBASE'

```
<!--more-->

## find
```
sudo -u admin find /home/admin /tmp /usr -name \*.log(多个目录去找)
find . -iname \*.txt(大小写都匹配)
find . -type d(当前目录下的所有子目录)
find /usr -type l(当前目录下所有的符号链接)
find /usr -type l -name "z*" -ls(符号链接的详细信息 eg:inode,目录)
find /home/admin -size +250000k(超过250000k的文件，当然+改成-就是小于了)
find /home/admin f -perm 777 -exec ls -l {} \; (按照权限查询文件)
find /home/admin -atime -1  1天内访问过的文件
find /home/admin -ctime -1  1天内状态改变过的文件    
find /home/admin -mtime -1  1天内修改过的文件
find /home/admin -amin -1  1分钟内访问过的文件
find /home/admin -cmin -1  1分钟内状态改变过的文件    
find /home/admin -mmin -1  1分钟内修改过的文件

```

## top
top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了
```
ps -ef | grep java
top -H -p  加上pid
```
获得线程10进制转16进制后jstack去抓看这个线程到底在干啥


# 排查利器
## btrace 用来监控数据 
生产环境&预发的排查问题  
https://github.com/btraceio/btrace  

例如:  
1、查看当前谁调用了ArrayList的add方法，同时只打印当前ArrayList的size大于500的线程调用栈
```
@OnMethod(clazz = "java.util.ArrayList", method="add", location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/"))
public static void m(@ProbeClassName String probeClass, @ProbeMethodName String probeMethod, @TargetInstance Object instance, @TargetMethodOrField String method) {
   if(getInt(field("java.util.ArrayList", "size"), instance) > 479){
       println("check who ArrayList.add method:" + probeClass + "#" + probeMethod  + ", method:" + method + ", size:" + getInt(field("java.util.ArrayList", "size"), instance));
       jstack();
       println();
       println("===========================");
       println();
   }
}
```
2、监控当前服务方法被调用时返回的值以及请求的参数
```
@OnMethod(clazz = "com.taobao.sellerhome.transfer.biz.impl.C2CApplyerServiceImpl", method="nav", location = @Location(value = Kind.RETURN))
public static void mt(long userId, int current, int relation, String check, String redirectUrl, @Return AnyType result) {
   println("parameter# userId:" + userId + ", current:" + current + ", relation:" + relation + ", check:" + check + ", redirectUrl:" + redirectUrl + ", result:" + result);
}
```

# JVM自带监控
## JPS
查看当前运行的java进程
```
jps 

```

## jstack
查看当前线程信息 检查是否死锁
```
常规用法:
jstack pid

查看native+java栈:
jstack -m pid


```

## jinfo
查看系统启动的参数
```
jinfo -flags pid

```

## jmap
1.查看堆的情况
```
jmap -heap pid
```
2.查看dump
```
jmap -dump:live,format=b,file=/tmp/heap2.bin 2815

jmap -dump:format=b,file=/tmp/heap3.bin 2815

```
3.看看堆都被谁占了
```
jmap -histo 2815 | head -10
```

## jstat
监控GC的内存分配区域
```
jstat -gcutil pid 1000 

```

# 设置JVM参数
## 你的类到底是从哪个文件加载进来的？
```
-XX:+TraceClassLoading
结果形如[Loaded java.lang.invoke.MethodHandleImpl$Lazy from D:\programme\jdk\jdk8U74\jre\lib\rt.jar]
```
## 应用挂了输出dump文件
```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/admin/logs/java.hprof

```