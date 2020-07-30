---
title: K8S笔记集群(二)
date: 2020-07-30 09:26:23
tags: [K8S笔记]
---

# K8S笔记集群(二)
采用`kwbewdm`的安装方案

# 暂时不会安装 先忽略了
安装步骤在 `/hexo/text/k8s讲义`里

### 第一步 linux 指定主机名
```
hostnamectl set-hostname k8s-master1

//其他节点ip映射
vim /etc/host 
->192.168.66.20 k8s-node01
```

### 第二步 安装依赖
```
yum install conntrack ntpdate  ipvsadm ipset jq iptables  curl systat libseccomp wget vim net-tools git
```
<!--more-->
### 第三步 设置防火墙为Iptables 并设置空规则
```
systemctl stop firewalld && systemctl disable firewalld
 yum -y install iptables-services && systemctl start iptables && systemctl enable iptables && iptables -F && service iptables save 
```

### 第四步 关闭SELINUX 不然pod可能运行在虚拟内存里
```
swapoff -a && sed -i `/ swap / s/^\(.*\)$/#\1/g` /etc/fstab
serenforce 0 && sed -i `s/^SELINUX=.*/SELINUX=disabled/` /etc/selinux/config

```

### 第五步 调整内核参数,对于k8s
```

```