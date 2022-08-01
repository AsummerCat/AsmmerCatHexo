---
title: MapReduce切面机制和MapTask并行度决定机制四
date: 2022-08-01 16:19:36
tags: [大数据,hadoop,MapReduce]
---
# MapReduce切面机制和MapTask并行度决定机制

1.一个Job的Map阶段并行度由客户端在提交job时候的切片数决定
(切出来每片数据都在本地处理,不跨越到其他的DataNode节点)

2.每个Split切片分配一个MapTask并行实例处理

3.默认情况下 ,切片大小=BlockSize (HDFS的块大小)

4.切片时候不考虑数据集整体,而是逐个针对每一个文件单独切片
(也就是不管你提交多少个文件,都是按照每个文件单独进行切片处理)

<!--more-->

## 如何修改job的切片规则
```
//如果不设置InputFormat,默认使用的是TextInputFormat
job.setInputFormatClass(CombineTextInputFormat.class);

//虚拟存储切片最大值设置4M
CombineTextInputFormat.setMaxInputSplitSize(job,4194304);
//4M
```



## FileInputFormat切片原理

```
1.程序先找到你数据存储的目录

2.开始遍历处理(规划切片)下的每一个文件

3.遍历第一个文件ss.txt
   3.1 获取文件大小fs.sizeOf
   (ss.txt)
   3.2 计算切片大小 computeSplitSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=123M
   3.3 默认情况下,切片大小=hdfs的块大小
   3.4 开始切,行程第一个切片:ss.txt-0:128M(每次切片时,都要判断切完剩下的部分是否大于块的1.1倍,不大于就不形成一个新的切片)
   3.5将切片信息写到一个切片规划文件中(HDFS输出路径中有个*.split的文件)
   3.6整个切片的核心过程在getSplit()方法中完成
   3.7 InputSplit只记录了切片的元数据信息,比如其实位置,长度,及其所在的节点列表等
   
4. 提交切片规范化文件到YARN上,YARN上的MrAppMatser就可以根据切片规划文件计算开启MapTask个数;

5.开始任务调度计算 
```

# CombineTextInputFormat切面机制 (生产环境对于小文件经常使用)
1.虚拟存储切片最大值设置
```
CombineTextInputFormat.setMaxInputSplitSize(job,4194304);
//4M
```

2.生成切片过程包括: 虚拟存储过程和切片过程两部分
```
可以将多个小文件从逻辑上规划到一个切片中,
这样多个小文件就可以交给一个MapTask处理;
```
有个虚拟切割存储的过程
```
CombineTextInputFormat.setMaxInputSplitSize(job,4194304);
//4M
```
流程:
```
假设 4个文件
a.txt 1.7M
b.txt 5.1M
c.txt 3.4M
d.txt 6.8M

虚拟存储分配后:
先按照小文件的名称进行一个排序 然后,
小于MaxInputSplitSize划分一块,但是
如果大于MaxInputSplitSize并且小于2*MaxInputSplitSize的话,平均切分两块

1.7M
2.55M
2.55M
3.4M
3.4M
3.4M

最终切面过程:

如果虚拟存储分配过后的文件不大于MaxInputSplitSize,
则和下一个虚拟存储文件进行合并,共同形成一个切片;

最终会行程3个切片,大小分别为:

(1.7+2.55)M,
(2.55+3.4)M,
(3.4+3.4)M
```