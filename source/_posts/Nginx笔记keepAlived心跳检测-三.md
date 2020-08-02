---
title: Nginx笔记keepAlived心跳检测(三)
date: 2020-08-02 15:41:55
tags: [Nginx笔记]
---

# nginx笔记keepAlived心跳检测(三)
## 安装keepAlive软件
路径:`/etc/keepalived`  
配置文件: `/etc/keepalived/keepalived.conf`  
日志文件: `var/log/messages`
```
# sudo yum install keepalived -y
# sudo systemctl start keepalived.service
# sudo systemctl enable keepalived.service
```
默认你已经安装nginx
```
# sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# sudo yum install -y nginx
# sudo systemctl start nginx.service
# sudo systemctl enable nginx.service
```
<!--more-->
## 编写 nginx 服务存活检测脚本
### 创建脚本
`# vim /usr/bin/check_nginx_alive.sh`

脚本内容:
检测nginx是否挂了 挂了重启一次 还是失败关闭keepalive
```
#!/bin/sh
 
PATH=/bin:/sbin:/usr/bin:/usr/sbin
 
A=`ps -C nginx --no-header |wc -l`
echo 'nginx存活台数'$A
if [ $A -eq 0 ]
   then
     sleep 2
     
    if [ps -C nginx --no-header |wc -l` -eq 0]
     then 
     echo 'nginx server is died'
     killall keepalived
     # 或者
     systemctl stop keepalived.service
    fi
fi
```
然后将该脚本配置到keepalived.conf中可以实现启动就监听nginx
```
## 监听nginx
vrrp_script chk_http_port{
    script "/usr/bin/check_nginx_alive.sh"  # 脚本地址
    interval 2 # 检测脚本执行的间隔
    weight 2   # 比重
}
```

### [脚本具体写法](https://linjingc.top/2019/04/19/keepAlive-nginx%E5%AE%9E%E7%8E%B0%E8%99%9A%E6%8B%9Fvip/)

## keepalived具体配置内容
```
global_defs {     # 全局配置
   notification_email {   #配置宕机的时候发送邮件
     acassen@firewall.loc    #收件人邮箱1
     failover@firewall.loc   #收件人邮箱2
     sysadmin@firewall.loc   #收件人邮箱3
   }
   notification_email_from Alexandre.Cassen@firewall.loc # 邮件发送人
   smtp_server 192.168.200.1  # 邮件服务器地址
   smtp_connect_timeout 30   # 超时时间
   router_id LVS_DEVEL       # 路由id,多个keepalived集群需要一致  ,修改host 127.0.0.1->指向这个LVS_DEVEL
   vrrp_skip_check_adv_addr   #默认不跳过检查
   # vrrp_strict    # 这个需要注释掉 严格遵守VRRP协议
   vrrp_garp_interval 0  # 单位秒 ->网卡上每组gratuitous arp消息之间的延迟
   vrrp_gna_interval 0  # 单位秒 ->网卡上每组 Na消息之间的延迟
}

## 监听nginx
vrrp_script chk_http_port{
    script "/usr/bin/check_nginx_alive.sh"  # 脚本地址
    interval 2 # 检测脚本执行的间隔
    weight 2   # 比重
}

# vrrp 实例配置 集群配置除了state,priority不一样其他都一样
vrrp_instance VI_1 {
    state MASTER    # 服务器状态 MASTER是主节点,BACKUP是备用节点 ,主节点的优先级要大于备用节点
    interface eth0  # 通信端口 可以通过ip addr 来查看
    virtual_router_id 51  # vrrp实例ID,keeplived集群,实例id必须一致
    priority 100    # 优先级
    advert_int 1    # 心跳间隔
    authentication {  # 服务器之间通信密码
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {  # (主备机一致)自定义虚拟ip 暴露出去给用户访问的
        192.168.200.99
    }
}


```