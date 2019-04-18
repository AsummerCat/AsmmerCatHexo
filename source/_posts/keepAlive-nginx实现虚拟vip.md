---
title: keepAlive+nginx实现虚拟vip
date: 2019-04-19 07:31:51
tags: [nginx,keepalive]
---

# keepAlive+nginx实现虚拟vip

# keepAlive

keepAlive是一个可以产生虚拟  v  ip的工具
避免单点故障 

类似于模拟一个节点 

# 引自知乎.灯下黑.的一段理解

VIP是也是实实在在的IP地址，跟一般的IP地址是一样的，  
自己的理解是 ”可以漂移的IP地址“（不官方）。  
1、要配置keepalived服务，需要事先需要准备三个IP地址。主服务器上的IP，备服务器上的IP，对外提供服务的VIP。  
主、备服务机器的IP直接配置到各自的网卡上。VIP写到配置文件里面，keepalived会将VIP配置到主服务器的网卡上。  
2、用户访问的是VIP。只有主服务器的keepalived会将VIP配置到网卡上，容灾之后，主变备：keepalived将VIP从网卡上移走。   
备变主：keepalived将VIP配置到网卡上。3、用户访问域名如何防止单点瘫痪：配好DNS，{域名:VIP}。用户访问该域名，经过解析，拿到VIP。  
此时，数据包会走上面主备服务器中的主服务器，  
如果主服务器瘫痪，备服务器会自动接过VIP，备变主，对外提供服务。整个过程，对于用户来说是透明的，用的是同样的域名，解析到的是同样的VIP。  

。

<!--more-->

## 安装keepAlive软件

```
# sudo yum install keepalived -y
# sudo systemctl start keepalived.service
# sudo systemctl enable keepalived.service
```

## 默认你已经安装nginx

```
# sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
# sudo yum install -y nginx
# sudo systemctl start nginx.service
# sudo systemctl enable nginx.service
```

## 编写 nginx 服务存活检测脚本

## 创建脚本

```
# vim /usr/bin/check_nginx_alive.sh
```

## 写入脚本

下面这个脚本的意思是

->查询出当前nginx 去除头信息后还有几个进程,
判断 进程数是否等于0 如果是杀死所有keepAlived的进程

```
#!/bin/sh
 
PATH=/bin:/sbin:/usr/bin:/usr/sbin
 
A=`ps -C nginx --no-header |wc -l`
 
if [ $A -eq 0 ]
   then
     echo 'nginx server is died'
     killall keepalived
fi


```

### 赋予执行权限

```
 chmod +x /usr/bin/check_nginx_alive.sh
```

## 配置 keepalive

(因为这边需要实现两台机子实现虚拟vip )

### 机器A

```
vrrp_script check_nginx_alive {
    script "/usr/bin/check_nginx_alive.sh"
    interval 3
    weight -10
}
global_defs {
    ## 设置lvs的id，在一个网络内唯一
    router_id LVS_DEVEL
}
 
vrrp_instance VI_1 {
    ## 主机配置，从机为BACKUP
    state MASTER
    ## 网卡名称
    interface ens37
    virtual_router_id 51
    ## 权重值,值越大，优先级越高，backup设置比master小,这样就能在master宕机后讲backup变为master,而master回复后就可以恢复.
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        ## 同一网段虚拟IP
        192.168.1.100
    }
    track_script {
        check_nginx_alive
    }
 
}
 
virtual_server 192.168.1.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
 
    real_server 192.168.1.9 80 {
        weight 1
        TCP_CHECK{
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}

```

### 机器B

```
vrrp_script check_nginx_alive {
    script "/usr/bin/check_nginx_alive.sh"
    interval 3
    weight -10
}
global_defs {
    ## 设置lvs的id，在一个网络内唯一
    router_id LVS_DEVEL
}
 
vrrp_instance VI_1 {
    ## 主机配置，从机为BACKUP
    state BACKUP
    ## 网卡名称
    interface ens37
    virtual_router_id 51
    ## 权重值,值越大，优先级越高，backup设置比master小,这样就能在master宕机后讲backup变为master,而master回复后就可以恢复.
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        ## 同一网段虚拟IP
        192.168.1.100
    }
    track_script {
        check_nginx_alive
    }
 
}
 
virtual_server 192.168.1.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP
 
    real_server 192.168.1.8 80 {
        weight 1
        TCP_CHECK{
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}

```

这里需要注意的是 虚拟ip 是固定的 不管多少台机子virtual_ipaddress 

注解： 虚拟 IP 为 192.168.1.100，A 机器 IP 为 192.168.1.9，B 机器 IP 为 192.168.1.8

```
               A 为 Master，B 为 Slave，A 优先级（100）高于 B 优先级（90），
```

## 这样子就已经创建好两个nginx+keepalive实现虚拟vip

## 接着修改 Nginx 主页，便于追溯主机

```
# vim /usr/share/nginx/html/index.html
```

## 最后重启两台服务器的keepalive 

因为我们这边修改了配置文件

```
 systemctl restart keepalived
```

# 最后注意事项

```
# systemctl start nginx
# systemctl restart keepalived

         本实验验证了 VIP 的自动漂移，基本实现了nginx 的主备自动切换

         值得注意的是，修复失败的服务后，

         必须重启所在机器的keepalive服务，否则keepalive是无法感知到服务恢复的！！！


```