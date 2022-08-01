---
title: django的学习-HTML模板语法(二)
date: 2019-12-17 15:25:19
tags: [python,django]
---

# 项目结构

[demo地址](https://github.com/AsummerCat/gogogo)

[官方文档地址](https://docs.djangoproject.com/en/2.2/ref/templates/builtins/)

# html模板语法

## py输出页面

    context = {}
    return render(request, 'hello.html', context)

## 直接渲染

```
html =>        {{ hello  }}
py   =>        context['hello'] = 'Hello World!'
```

<!--more-->

## if标签

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

## for循环

```
<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% endfor %}
</ul>
```

您可以使用反向遍历列表 。

```
`{% for obj in list reversed %}`
for循环设置循环中可用的许多变量：

变量	描述
forloop.counter	循环的当前迭代（1索引）
forloop.counter0	循环的当前迭代（0索引）
forloop.revcounter	从循环末尾开始的迭代次数（1索引）
forloop.revcounter0	从循环末尾开始的迭代次数（0索引）
forloop.first	如果这是第一次循环，则为真
forloop.last	如果这是最后一次循环，则为true
forloop.parentloop	对于嵌套循环，这是围绕当前循环的循环
```

### `for…empty`

```
如果给定数组为空或找不到，则`for`标记可以带有一个可选子句，其文本将显示：`{% empty %}`

<ul>
{% for athlete in athlete_list %}
    <li>{{ athlete.name }}</li>
{% empty %}
    <li>Sorry, no athletes in this list.</li>
{% endfor %}
</ul>
```

## fequal/ifnotequal 标签  对比值

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

## 注释标签

```
{# 这是一个注释 #}
```

## 过滤器

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

## include 标签

下面这个例子都包含了 `nav.html `模板：

```
`{% include %} `标签允许在模板中包含其它的模板的内容。

{% include "nav.html" %}
```

## 模板继承

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

## 获取当前用户

```
{{ request.user }}
```

如果登陆就显示内容，不登陆就不显示内容：

```
{% if request.user.is_authenticated %}
    {{ request.user.username }}，您好！
{% else %}
    请登陆，这里放登陆链接
{% endif %}
```

## **获取当前网址**

```
{{ request.path }}
```

## **获取当前 GET 参数**

```
{{ request.GET.urlencode }}
```

##  **合并到一起用的一个例子**

```
<a href="{{ request.path }}?{{ request.GET.urlencode }}&delete=1">当前网址加参数 delete</a>
```

比如我们可以判断 delete 参数是不是 1 来删除当前的页面内容。

## `cycle` 用来循环中 标记 类似于 表格中隔行切换样式

每次遇到此标签时，都会产生其参数之一。第一个参数在第一次遇到时产生，第二个参数在第二次遇到时产生，依此类推。一旦所有参数用尽，标记将循环到第一个参数并再次产生它。

此标记在循环中特别有用：

```
{% for o in some_list %}
    <tr class="{% cycle 'row1' 'row2' %}">
        ...
    </tr>
{% endfor %}
```

第一次迭代生成的HTML引用了class `row1`，第二次引用了，`row2`第三次 引用`row1`了，依次类推。

您也可以使用变量。例如，如果您有两个模板变量 `rowvalue1`和`rowvalue2`，则可以在它们的值之间交替如下：

```
{% for o in some_list %}
    <tr class="{% cycle rowvalue1 rowvalue2 %}">
        ...
    </tr>
{% endfor %}
```

循环中包含的变量将被转义。您可以使用以下方法禁用自动转义：

```
{% for o in some_list %}
    <tr class="{% autoescape off %}{% cycle rowvalue1 rowvalue2 %}{% endautoescape %}">
        ...
    </tr>
{% endfor %}
```

在某些情况下，您可能希望引用循环的当前值而不前进到下一个值。为此，只需使用“ as” 为标签命名，如下所示：

```
`{% cycle %}`
```

```
{% cycle 'row1' 'row2' as rowcolors %}
```

从那时起，您可以通过将循环名称作为上下文变量引用，在模板中的任意位置插入循环的当前值。如果要独立于原始`cycle`标签将循环移动到下一个值，则 可以使用另一个`cycle`标签并指定变量的名称。因此，以下模板：

```
<tr>
    <td class="{% cycle 'row1' 'row2' as rowcolors %}">...</td>
    <td class="{{ rowcolors }}">...</td>
</tr>
<tr>
    <td class="{% cycle rowcolors %}">...</td>
    <td class="{{ rowcolors }}">...</td>
</tr>
```

将输出:

```
<tr>
    <td class="row1">...</td>
    <td class="row1">...</td>
</tr>
<tr>
    <td class="row2">...</td>
    <td class="row2">...</td>
</tr>
```

## 获取时间 `now`

使用根据给定字符串的格式显示当前日期和/或时间。这样的字符串可以包含格式说明符，如[`date`](https://docs.djangoproject.com/zh-hans/2.2/ref/templates/builtins/#std:templatefilter-date)过滤器部分所述。

举例说明：

```
It is {% now "jS F Y H:i" %}
```

## `regroup`[¶](https://docs.djangoproject.com/zh-hans/2.2/ref/templates/builtins/#regroup)  分组

通过通用属性将相似对象列表重新组合。

说法是：这复杂的标签，最好是用一个例子来说明`cities` 是含有字典代表城市的名单`"name"`， `"population"`和`"country"`键：

```
cities = [
    {'name': 'Mumbai', 'population': '19,000,000', 'country': 'India'},
    {'name': 'Calcutta', 'population': '15,000,000', 'country': 'India'},
    {'name': 'New York', 'population': '20,000,000', 'country': 'USA'},
    {'name': 'Chicago', 'population': '7,000,000', 'country': 'USA'},
    {'name': 'Tokyo', 'population': '33,000,000', 'country': 'Japan'},
]
```

...并且您想要显示按国家/地区排序的分层列表，如下所示：

- 印度
  - 孟买：19,000,000
  - 加尔各答：15,000,000
- 美国
  - 纽约：20,000,000
  - 芝加哥：7,000,000
- 日本
  - 东京：33,000,000

您可以使用标记按国家对城市列表进行分组。下面的模板代码片段将完成此任务：

```
`{% regroup %}`
```



```
{% regroup cities by country as country_list %}

<ul>
{% for country in country_list %}
    <li>{{ country.grouper }}
    <ul>
        {% for city in country.list %}
          <li>{{ city.name }}: {{ city.population }}</li>
        {% endfor %}
    </ul>
    </li>
{% endfor %}
</ul>
```



```
让我们来看这个例子。接受三个参数：要重新分组的列表，分组依据的属性以及结果列表的名称。在这里，我们通过 属性将列表重新分组并调用result 。`{% regroup %}``cities``country``country_list`

`{% regroup %}`生成**组对象**的列表（在这种情况下为`country_list`） 。组对象是具有两个字段的实例 ：[`namedtuple()`](https://docs.python.org/3/library/collections.html#collections.namedtuple)

- `grouper` -分组的项目（例如，字符串“印度”或“日本”）。
- `list` -该组中所有项目的列表（例如，所有国家/地区为“印度”的城市的列表）。

由于产生对象，因此您还可以将前面的示例编写为：`{% regroup %}`[`namedtuple()`](https://docs.python.org/3/library/collections.html#collections.namedtuple)

{% regroup cities by country as country_list %}

<ul>
{% for country, local_cities in country_list %}
    <li>{{ country }}
    <ul>
        {% for city in local_cities %}
          <li>{{ city.name }}: {{ city.population }}</li>
        {% endfor %}
    </ul>
    </li>
{% endfor %}
</ul>
```

请注意，请勿订购其输入！我们的示例依赖于这样的事实，即列表是按顺序排列的。如果该列表*未按*排序，则重组将仅显示单个国家的多个组。例如，假设列表设置为此（请注意，这些国家未分组在一起）：

```
`{% regroup %}``cities``country``cities``country``cities`

cities = [
    {'name': 'Mumbai', 'population': '19,000,000', 'country': 'India'},
    {'name': 'New York', 'population': '20,000,000', 'country': 'USA'},
    {'name': 'Calcutta', 'population': '15,000,000', 'country': 'India'},
    {'name': 'Chicago', 'population': '7,000,000', 'country': 'USA'},
    {'name': 'Tokyo', 'population': '33,000,000', 'country': 'Japan'},
]
```

使用的输入`cities`，上面的示例模板代码将产生以下输出：

```
`{% regroup %}`
```



- 印度
  - 孟买：19,000,000
- 美国
  - 纽约：20,000,000
- 印度
  - 加尔各答：15,000,000
- 美国
  - 芝加哥：7,000,000
- 日本
  - 东京：33,000,000

解决此难题的最简单方法是，在您的视图代码中确保根据要显示的数据对数据进行排序。

## `spaceless`[¶](https://docs.djangoproject.com/zh-hans/2.2/ref/templates/builtins/#spaceless)  删除空格

删除HTML标记之间的空格。这包括制表符和换行符。

用法示例：

```
{% spaceless %}
    <p>
        <a href="foo/">Foo</a>
    </p>
{% endspaceless %}
```

此示例将返回以下HTML：

```
<p><a href="foo/">Foo</a></p>
```

*标签*之间只有空格，而标签和文本之间没有空格。在此示例中，周围的空间`Hello`不会被剥离：

```
{% spaceless %}
    <strong>
        Hello
    </strong>
{% endspaceless %}
```

## `url`[¶](https://docs.djangoproject.com/zh-hans/2.2/ref/templates/builtins/#url) urls.py 指定动态匹配路径名称

返回与给定视图和可选参数匹配的绝对路径引用（不带域名的URL）。结果路径中的任何特殊字符都将使用进行编码[`iri_to_uri()`](https://docs.djangoproject.com/zh-hans/2.2/ref/utils/#django.utils.encoding.iri_to_uri)。

通过在模板中对URL进行硬编码，这是一种输出链接而不违反DRY原理的方法：

```
{% url 'some-url-name' v1 v2 %}
```

第一个参数是[URL模式名称](https://docs.djangoproject.com/zh-hans/2.2/topics/http/urls/#naming-url-patterns)。它可以是带引号的文字或任何其他上下文变量。其他参数是可选的，并且应为以空格分隔的值，这些值将用作URL中的参数。上面的示例显示了传递位置参数。或者，您可以使用关键字语法：

```
{% url 'some-url-name' arg1=v1 arg2=v2 %}
```