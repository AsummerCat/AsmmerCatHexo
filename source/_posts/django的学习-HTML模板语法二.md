---
title: django的学习-HTML模板语法二
date: 2019-12-17 15:25:19
tags: [python,django]
---

# 项目结构

[demo地址](https://github.com/AsummerCat/gogogo)



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

### fequal/ifnotequal 标签  对比值

```
`{% ifequal %} `标签比较两个值，当他们相等时，显示在` {% ifequal %} `和` {% endifequal %} `之中所有的值。

{% ifequal user currentuser %}
    <h1>Welcome!</h1>
{% endifequal %}
```

```
和 `{% if %} `类似，` {% ifequal %} `支持可选的` {% else%} `标签：8

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

`{{ name }} `变量被过滤器 lower 处理后，文档大写转换文本为小写。

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

下面这个例子都包含了 `nav.html `模板：

```
`{% include %} `标签允许在模板中包含其它的模板的内容。

{% include "nav.html" %}
```

### 模板继承

模板可以用继承的方式来实现复用。

接下来我们先创建之前项目的` templates` 目录中添加 `base.html` 文件，代码如下：

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

```
以上代码中，名为 mainbody 的 block 标签是可以被继承者们替换掉的部分。

所有的 `{% block %} `标签告诉模板引擎，子模板可以重载这些部分。

hello.html 中继承 base.html，并替换特定 block，hello.html 修改后的代码如下：
```



```
{%extends "base.html" %}
 
{% block mainbody %}
<p>继承了 base.html 文件</p>
{% endblock %}
```

