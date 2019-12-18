---
title: django的学习-了解基础部分一
date: 2019-12-16 14:07:42
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

## html模板语法

## py输出页面

    context = {}
    return render(request, 'hello.html', context)
### 直接渲染

```
html =>        {{ hello  }}
py   =>        context['hello'] = 'Hello World!'
```

### if标签

```
{% if condition %}
     ... display
{% endif %}
```

or

```
{% if condition1 %}
   ... display 1
{% elif condition2 %}
   ... display 2
{% else %}
   ... display 3
{% endif %}
```

根据条件判断是否输出。if/else 支持嵌套。

{% if %} 标签接受 and ， or 或者 not 关键字来对多个变量做判断 ，或者对变量取反（ not )，例如：

```
{% if athlete_list and coach_list %}
     athletes 和 coaches 变量都是可用的。
{% endif %}
```

### for循环

```
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% endfor %}
</ul>
```

给标签增加一个 reversed 使得该列表被反向迭代：

```
{% for athlete in athlete_list reversed %}
...
{% endfor %}
```

py写法

```
 context['athlete_list'] = ['abcd', 786, 2.23, 'runoob', 70.2]
```

### fequal/ifnotequal 标签  对比值

{% ifequal %} 标签比较两个值，当他们相等时，显示在 {% ifequal %} 和 {% endifequal %} 之中所有的值。

```
{% ifequal user currentuser %}
    <h1>Welcome!</h1>
{% endifequal %}
```

和 {% if %} 类似， {% ifequal %} 支持可选的 {% else%} 标签：8

```
{% ifequal section 'sitenews' %}
    <h1>Site News</h1>
{% else %}
    <h1>No News Here</h1>
{% endifequal %}
```

### 注释标签

```
{# 这是一个注释 #}
```

### 过滤器

模板过滤器可以在变量被显示前修改它，过滤器使用管道字符，如下所示：

```
{{ name|lower }}
```

{{ name }} 变量被过滤器 lower 处理后，文档大写转换文本为小写。

过滤管道可以被* 套接* ，既是说，一个过滤器管道的输出又可以作为下一个管道的输入：

```
{{ my_list|first|upper }}
```

以上实例将第一个元素并将其转化为大写。

有些过滤器有参数。 过滤器的参数跟随冒号之后并且总是以双引号包含。 例如：

```
{{ bio|truncatewords:"30" }}
```

这个将显示变量 bio 的前30个词。

其他过滤器：

- addslashes : 添加反斜杠到任何反斜杠、单引号或者双引号前面。

- date : 按指定的格式字符串参数格式化 date 或者 datetime 对象，实例：

  ```
  {{ create_time | date:"Y-m-d H:i:s" }}
  ```

- length : 返回变量的长度。

### include 标签

{% include %} 标签允许在模板中包含其它的模板的内容。

下面这个例子都包含了 nav.html 模板：

```
{% include "nav.html" %}
```

### 模板继承

模板可以用继承的方式来实现复用。

接下来我们先创建之前项目的 templates 目录中添加 base.html 文件，代码如下：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>父类</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p>父类的模块</p>
    {% block mainbody %}
       <p>original</p>
    {% endblock %}
</body>
</html>
```

以上代码中，名为 mainbody 的 block 标签是可以被继承者们替换掉的部分。

所有的 {% block %} 标签告诉模板引擎，子模板可以重载这些部分。

hello.html 中继承 base.html，并替换特定 block，hello.html 修改后的代码如下：

```
{%extends "base.html" %}
 
{% block mainbody %}
<p>继承了 base.html 文件</p>
{% endblock %}
```

