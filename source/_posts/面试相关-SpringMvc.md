---
title: 面试相关-SpringMvc
date: 2020-02-04 00:30:31
tags: [面试相关]
---

# SpringMvc

## 工作流程

```
1.客户端（浏览器）发送请求，直接请求到DispatcherServlet。
2.DispatcherServlet根据请求信息调用HandlerMapping(处理器映射器)，解析请求对应的Handler。
3.解析到对应的Handler（也就是我们平常说的Controller控制器）。
4.HandlerAdapter(处理器适配器)会根据Handler来调用真正的处理器来处理请求和执行相对应的业务逻辑。
5.处理器处理完业务后，会返回一个ModelAndView对象，Model是返回的数据对象，View是逻辑上的View。
6.ViewResolver会根据逻辑View去查找实际的View。(视图解析)
7.DispatcherServlet把返回的Model传给View（视图渲染）。
8.把View返回给请求者（浏览器）。
```

<!--more-->

