---
title: Nginx笔记其他配置(四)
date: 2020-08-02 15:42:19
tags: [Nginx笔记]
---

# nginx笔记其他配置(四)

## 开启日志缓存
因为 高流量的情况下 ,每个日志都是打开文件->写入日志->关闭文件    
开启缓存 这样可以加快速度

ps:不建议开启 ->降低io 占用内容
```
开启日志缓存: 最大缓存数,无效时间(淘汰策略),min_user(低于三次访问就删除缓存),有效时间(一分钟检查一次是否更新)

open_log_file_cache max=1000 inactive=20s min_uses=3 valid=1m;
```
<!--more-->
```
缓存可以放的位置:
http ,server,location
```
服务器默认情况下是关闭缓存的
```
open_log_file_cache off;
```

## 随机主页
```
在location节点中加入
location/{
    //app文件夹下放入多个html
    root /app;
    //随机访问
    random_index on;
}

```

## nginx访问控制
```
ngx_http_limit_req_module 请求数

ngx_http_limit_conn_module 连接数

基于主机
基于用户
```