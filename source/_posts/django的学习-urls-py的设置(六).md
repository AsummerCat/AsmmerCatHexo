---
title: django的学习-urls.py的设置(六)
date: 2019-12-19 17:08:25
tags: [python,django]
---

## 动态映射路径 (重点)

```python
# views.py
def add(request, a, b):
    c = int(a) + int(b)
    return HttpResponse(str(c))
 
 
# urls.py
urlpatterns = patterns('',
    path('add/<int:a>/<int:b>/', calc_views.add2, name='add'),
)
 
 
# template html
{% url 'add' 4 5 %}
```

这样网址上就会显示出：/add/4/5/ 这个网址，假如我们以后修改 urls.py 中的 

```
'add/<int:a>/<int:b>/'
```

这一部分，改成另的，比如：

```
'jia/<int:a>/<int:b>/'
```

这样，我们不需要再次修改模板，当再次访问的时候，网址会自动变成 /jia/4/5/



## 项目目录下配置（path方法

***这里的访问路径可以是127.0.0.1:8000/index/***   

![](/img/2019-12-19/1.png)

![](/img/2019-12-19/2.png)

<!--more-->

##  APP目录下配置（path方法)

```
这里的访问路径可以是127.0.0.1：8000/teacher/index/   （注意teacher不是APP名，而是crm/urls.py文件里面的path路径‘teacher/’）
```



![](/img/2019-12-19/3.png)

![](/img/2019-12-19/4.png)

![](/img/2019-12-19/5.png)



## path方法配置及传参

***这里的访问路径是127.0.0.1:8000/index/10000/  （可以传多个参数，参数与参数之间用 / 或者 -）***

![](/img/2019-12-19/6.png)

![](/img/2019-12-19/7.png)

## re_path方法配置及传参

***如果限制穿的参数为四位数，则用如图方法传递***

![](/img/2019-12-19/8.png)

## 传递额外参数

![](/img/2019-12-19/9.png)

##  url命名及重定向

### 重定向

![](/img/2019-12-19/10.png)

![](/img/2019-12-19/11.png)

