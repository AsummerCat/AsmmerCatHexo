---
title: ElasticSearch笔记运维集群之操作系统参数修改(四十)
date: 2020-09-06 22:08:17
tags: [elasticSearch笔记]
---

# 优化操作系统保证es稳定高效运行

## 在生产环境下的一些设置必须配置
1. 禁止swapping (内存转磁盘)
2. 确保拥有足够的虚拟内存
3. 确保拥有足够的线程数量
4. 调整进程的句柄数 ulimit 65536

<!--more-->

### 禁止swapping
三种方式
推荐的option是彻底禁用swap

1. 第一种:禁用所有的swapping file
```
临时禁止: 
   swap: swapoff -a
   
永久禁止:
修改'/etc/fstab`文件,然后将所有包含swap的行都注释掉

```
2. 第二种 配置swappiness
```
通过'sysctl' ,将vm.swappiness设置为1,
这可以减少linux内核swap的倾向
在正常情况下,就不会进行swap,
但是在紧急情况下,还是会进行swap操作
```

3. 第三种 启用bootstrap.memory lock
```
开启内存锁,将es进程的address space锁定在内存中,
阻止es内存被swap out到磁盘上去.
在'config/elasticsearch.yml'中,可以配置:
'boorstrap.memory_lock :true'

然后
'GET _nodes?filter_path=**.mlockall'
通过这一行命令可以检查'mlockall'是否开启了

如果发现显示为'false',那么意味着开启失败,
可能是因为启动es进程的用户没有权限去'lock memory',
通过以下方式进行授权:
'ulimit -l unlimited '
```



### 确保拥有足够的虚拟内存
需要提升`mmap count`的限制:  
```
临时修改:
sysctl -w vm.max_map_count=262144

永久修改:
打开'/etc/sysctl.conf'
将'vm.max_map_count'的值修改一下,然后重启
使用'sysctl vm.max_map_count'来验证一下数值是否修改成功
```




### 确保拥有足够的线程数量
要确保es用户能创建的最大线程数量至少在2048以上
```
临时修改:
 ulimit -u 2048

永久修改:
在'/etc/security/limits.conf'中
设置'nproc'为2048
```



### 调整进程的句柄数 ulimit
在`/etc/security/limits.conf`,可以配置系统设置

1.临时修改
```
sudo ulimit -n 65536
```
2.永久修改
```
vi /etc/security/limits.conf

加入
elasticsearch -nofile 65536

```