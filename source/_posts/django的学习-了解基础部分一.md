---
title: django的学习-了解基础部分一
date: 2019-12-17 15:18:19
tags: [python,django]
---

# 项目结构

[demo地址](https://github.com/AsummerCat/gogogo)

创建完成后我们可以查看下项目的目录结构：

```
$ cd HelloWorld/
$ tree
.
|-- HelloWorld
|   |-- __init__.py
|   |-- settings.py
|   |-- urls.py
|   `-- wsgi.py
`-- manage.py
```

目录说明：

- **HelloWorld:** 项目的容器。
- **manage.py:** 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。
- **HelloWorld/__init__.py:** 一个空文件，告诉 Python 该目录是一个 Python 包。
- **HelloWorld/settings.py:** 该 Django 项目的设置/配置。
- **HelloWorld/urls.py:** 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"。
- **HelloWorld/wsgi.py:** 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。

<!--more-->

## 构建一个基础项目

下载pharm按照流程构建 django项目

### 这边新建一个应用(重要)

```python
python manage.py startapp learn # learn 是一个app的名称
```

**把我们新定义的app加到settings.py中的****INSTALL_APPS****中**

### 创建一个controller层 helloWord.py

```python
# -*- coding: utf-8 -*-
from django.http import HttpResponse

"""
直接返回字符串
"""
def hello(request):
    return HttpResponse("hello world ! ")

```

### 然后在urls.py中配置该信息

```python
from gogogo.helloWord import hello

urlpatterns = [
    # 路径名称 ,路径方法,路径名称 用来反向显示
    path('admin/', admin.site.urls),
    path('hello/',hello, name='hello')
]

```

将路径映射进去 然后启动

这样一个简单的实例就启动了

注意: 一个方法 可以映射多个路径 还可以使用正则处理

# 使用http模板引擎渲染 类似jsp的方式

## 首先修改settings.py

```
 [os.path.join(BASE_DIR, '/templates')]      修改为->    'DIRS': [BASE_DIR+"/templates",]
 
 or 'DIRS': [os.path.join(BASE_DIR, 'HelloWorld/templates')],
```

## 在templates写一个html模板

`hello.html`

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试模板引擎</title>
</head>
<body>
<h1>{{ hello }}</h1>
</body>
</html>
```

## 编写controller

```python
from django.shortcuts import render

'''
这边使用模板
'''


def helloHtml(request):
    # 参数放入字典输出页面
    context = {}
    context['hello'] = 'Hello World!'
    return render(request, 'hello.html', context)

```



## 修改urls.py 映射路径

```
import gogogo.helloWord as gogo

urlpatterns = [
    # 路径名称 ,路径方法,路径名称 用来反向显示
    path('helloHtml/', gogo.helloHtml, name='helloHtml')
]
```

## 这样基本就完成映射了



# 运行方式

## 本地运行

```;python
python manage.py runserver 
```



## 其他电脑允许访问

```
python manage.py runserver 0.0.0.0:8000

 
监听机器上所有ip 8000端口，访问时用电脑的ip代替 127.0.0.1
```

### 

**在我们创建的项目里修改setting.py文件**

**ALLOWED_HOSTS = ['\*']  ＃在这里请求的host添加了\***

这样才允许其他ip访问