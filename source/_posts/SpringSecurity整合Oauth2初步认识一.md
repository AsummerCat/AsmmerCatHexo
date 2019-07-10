---
title: SpringSecurity整合Oauth2初步认识一
date: 2019-07-10 22:03:14
tags: [Oauth2,Security,springCloud]
---

# SpringSecurity整合Oauth2初步认识(一)

[demo地址](https://github.com/AsummerCat/oauthdemo)

# 介绍

关于oauth2，其实是一个规范，本文重点讲解[spring](http://lib.csdn.net/base/javaee)对他进行的实现，如果你还不清楚授权服务器，资源服务器，认证授权等基础概念，可以移步[理解OAuth 2.0 - 阮一峰](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)，这是一篇对于oauth2很好的科普文章。

## oauth 分为4种模式

- 授权码模式（authorization code）     //第三方登录 授权码模式使用到了回调地址
- 简化模式（implicit）                           //不常用
- 密码模式（resource owner password credentials）   //客户端需要获取到用户密码
- 客户端模式（client credentials）  只需要秘钥和id就好了

<!--more-->

## 指定参数

```java
客户端申请认证的URI，包含以下参数：

response_type：表示授权类型，必选项，此处的值固定为"code"
client_id：表示客户端的ID，必选项
redirect_uri：表示重定向URI，可选项
scope：表示申请的权限范围，可选项
state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。
```



## 整合oauth 分为两个服务 

- 认证授权服务器  (可以理解为你的密码需要认证后 返回给token)
- 资源服务器 (使用token 访问 你的服务)

## oauth 请求方式

### 授权码模式

```java
客户端 ->跳转第三方登录(认证) ->认证完毕回调 ->根据回调获取授权码code -> 发送授权码去认证服务器上获取token ->token获取完毕后 ->获取第三方用户信息->注册/登录 -> 进入系统
```

### 密码模式

```
客户端登录密码  ->服务端发送http请求 认证服务器 获取token ->token获取完毕之后 ->赋值给头消息->返回页面
```

- password模式：`http://localhost:8080/oauth/token? username=user_1&password=123456& grant_type=password&scope=select& client_id=client_2&client_secret=123456`，响应如下：

```java
{
    "access_token":"950a7cc9-5a8a-42c9-a693-40e817b1a4b0",
    "token_type":"bearer",
    "refresh_token":"773a0fcd-6023-45f8-8848-e141296cb3cb",
    "expires_in":27036,
    "scope":"select"
}
```



### 客户端模式

```
客户端发起认证请求->认证服务器 认证完毕 ->返回token   ->根据token访问资源
```

- client模式：`http://localhost:8080/oauth/token? grant_type=client_credentials& scope=select& client_id=client_1& client_secret=123456`，响应如下：

```java
{
    "access_token":"56465b41-429d-436c-ad8d-613d476ff322",
    "token_type":"bearer",
    "expires_in":25074,
    "scope":"select"
}
```

