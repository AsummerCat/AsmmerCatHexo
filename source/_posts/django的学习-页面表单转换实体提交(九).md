---
title: django的学习-页面表单转换实体提交(九)
date: 2019-12-20 14:39:24
tags: [python,django]
---

# 页面的表单 后台转换为实体接收

[demo地址]()

entityFromDto.py entityFromSumbit.py entityFromSumbit.html urls.py

类似 java的实体类 接收数据类型 映射关系



<!--more-->

## 创建实体类

```python
from django import forms
 
class AddForm(forms.Form):
    a = forms.IntegerField()
    b = forms.IntegerField()
```

## 创建视图

```python
# coding:utf-8
from django.shortcuts import render
from django.http import HttpResponse
 
# 引入我们创建的表单类
from .forms import AddForm
 
def index(request):
    if request.method == 'POST':# 当提交表单时
     
        form = AddForm(request.POST) # form 包含提交的数据
         
        if form.is_valid():# 如果提交的数据合法
            a = form.cleaned_data['a']
            b = form.cleaned_data['b']
            return HttpResponse(str(int(a) + int(b)))
     
    else:# 当正常访问时
        form = AddForm()
    return render(request, 'index.html', {'form': form})
```

## 加入html模板

模板中会自动加载实体的字段 因为 controller中 from =AddFrom()

```html
<form method='post'>
{% csrf_token %}
{{ form }}
<input type="submit" value="提交">
</form>
```

## 映射路径urls.py

```
    path('entityFromSumbit/', entityFromSumbit.entityFromSumbit,name="entityFromSumbit"),
```

**新手可能觉得这样变得更麻烦了，有些情况是这样的，但是 Django 的 forms 提供了：**

1. 模板中表单的渲染

2. 数据的验证工作，某一些输入不合法也不会丢失已经输入的数据。

3. 还可以定制更复杂的验证工作，如果提供了10个输入框，必须必须要输入其中两个以上，在 forms.py 中都很容易实现

   

也有一些将 Django forms 渲染成 Bootstrap 的插件，也很好用，很方便。