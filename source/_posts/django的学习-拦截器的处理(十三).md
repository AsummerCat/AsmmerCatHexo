---
title: django的学习-拦截器的处理(十三)
date: 2019-12-20 16:40:13
tags: [python,django]
---

# 拦截器的处理

## 拦截器的存放路径

**1.1 Django 1.9 和以前的版本：**

```
MIDDLEWARE_CLASSES = (
    'zqxt.middleware.BlockedIpMiddleware',
    ...其它的中间件
)
```

**1.2 Django 1.10 版本 更名为 MIDDLEWARE（单复同形），写法也有变化，详见 第四部分。**

***\*如果用 Django 1.10版本开发，部署时用 Django 1.9版本或更低版本，要特别小心此处。\****

```
MIDDLEWARE = (
    'zqxt.middleware.BlockedIpMiddleware',
    ...其它的中间件
)
```

Django 会从 MIDDLEWARE_CLASSES 或 MIDDLEWARE 中按照从上到下的顺序一个个执行中间件中的 process_request 函数，而其中 process_response 函数则是最前面的最后执行。

<!--more-->

## 拦截器写法

这边的话 如果ip在黑名单内 拦截请求 返回Forbidden

```
# -*- coding: utf-8 -*-
from django import http
from django.conf import settings
from django.utils.deprecation import MiddlewareMixin


class BlockedIpMiddleware(MiddlewareMixin):

    def process_request(self, request):
        if request.META['REMOTE_ADDR'] in getattr(settings, "BLOCKED_IPS", []):
            return http.HttpResponseForbidden('<h1>Forbidden</h1>')
        else:
            print("并未加入黑名单")

```

