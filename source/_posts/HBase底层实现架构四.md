---
title: HBase底层实现架构四
date: 2022-08-22 11:32:22
tags: [大数据,HBase]
---
# HBase底层实现架构

```
Master:
  主要进程,具体实现类HMaster
  通常部署在namenode上

ReginServer
  主要进程,具体实现类为HRegionServer
  通常部署在dataNdode上
```
<!--more-->
# master架构
![image](/img/2022-08-22/1.png)



# ReginServer架构
![image](/img/2022-08-22/2.png)
![image](/img/2022-08-22/3.png)

# HBase写流程
![image](/img/2022-08-22/4.png)
![image](/img/2022-08-22/5.png)
![image](/img/2022-08-22/6.png)

# HBase读流程
![image](/img/2022-08-22/7.png)
![image](/img/2022-08-22/8.png)

### 合并读取数据优化
![image](/img/2022-08-22/9.png)

### 文件合并
类似GC的过程
![image](/img/2022-08-22/10.png) 

 