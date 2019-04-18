---
title: nginx限流方案的实现
date: 2019-04-19 07:30:49
tags: nginx
---

# nginx限流方案的实现

# 暂时两种方式

- limit_req_zone (漏桶)
- limit_conn_zone (令牌桶)
- ngx_http_upstream_module ()

<!--more-->

## 漏桶

在秒杀限流那边 有简单的介绍 
大意就是 并发下 给服务器的请求是一定的

```
http{ 
 limit_conn_zone $binary_remote_addr zone=one:10m; 
 server 
 { 
   ...... 
  limit_conn one 10; 
  ...... 
 } 
} 

```

其中“limit_conn one 10”既可以放在server层对整个server有效，也可以放在location中只对单独的location有效。  
该配置表明：客户端的并发连接数只能是10个。  

## 令牌桶

简单来说就是 定时推送令牌 获取令牌才能推送请求给服务器 不然拒绝请求返回503

```
http{ 
 limit_req_zone $binary_remote_addr zone=req_one:10m rate=1r/s; 
 server 
 { 
   ...... 
  limit_req zone=req_one burst=120 nodelay; 
  ...... 
 } 
} 
```

其中“limit_req zone=req_one burst=120”既可以放在server层对整个server有效，也可以放在location中只对单独的location有效。    
rate=1r/s的意思是每个地址每秒只能请求一次，也就是说令牌桶burst=120一共有120块令牌，并且每秒钟只新增1块令牌，120块令牌发完后，多出来的请求就会返回503.。  

zone：定义共享内存区来存储访问信息， req_one:10m 表示一个大小为10M，名字为req_one的内存区域。1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息。  

nodelay 针对的是 burst 参数，burst=20 nodelay 表示这20个请求立马处理，不能延迟，相当于特事特办。不过，即使这20个突发请求立马处理结束，后续来了请求也不会立马处理。burst=20 相当于缓存队列中占了20个坑，即使请求被处理了，这20个位置这只能按 100ms一个来释放。  



## ngx_http_upstream_module

该模块有一个参数：max_conns可以对服务端进行限流  

```
upstream xxxx{ 
  
 server 127.0.0.1:8080 max_conns=10; 
  
 server 127.0.0.1:8081 max_conns=10; 
  
} 

```

