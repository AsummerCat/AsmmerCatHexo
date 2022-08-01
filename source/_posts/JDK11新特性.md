---
title: JDK11新特性
date: 2022-08-01 13:54:18
tags: [jdk新特性]
---

# JDK11新特性
## HTTP Client 标准化
Java 11 对 Java 9 中引入并在 Java 10 中进行了更新的 Http Client API 进行了标准化，在前两个版本中进行孵化的同时，Http Client 几乎被完全重写，并且现在完全支持异步非阻塞。

并且，Java 11 中，Http Client 的包名由 jdk.incubator.http 改为java.net.http，该 API 通过 CompleteableFuture 提供非阻塞请求和响应语义。使用起来也很简单，如下：
```
var request = HttpRequest.newBuilder()
    .uri(URI.create("https://javastack.cn"))
    .GET()
    .build();
var client = HttpClient.newHttpClient();

// 同步
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());

// 异步
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);
```
<!--more-->

## String 增强
Java 11 增加了一系列的字符串处理方法：
```
//判断字符串是否为空
" ".isBlank();//true
//去除字符串首尾空格
" Java ".strip();// "Java"
//去除字符串首部空格
" Java ".stripLeading();   // "Java "
//去除字符串尾部空格
" Java ".stripTrailing();  // " Java"
//重复字符串多少次
"Java".repeat(3);             // "JavaJavaJava"
//返回由行终止符分隔的字符串集合。
"A\nB\nC".lines().count();    // 3
"A\nB\nC".lines().collect(Collectors.toList());
```

## Optional 增强
新增了empty()方法来判断指定的 Optional 对象是否为空。
```
var op = Optional.empty();
System.out.println(op.isEmpty());//判断指定的 Optional 对象是否为空
```

