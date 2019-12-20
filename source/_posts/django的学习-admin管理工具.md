---
title: django的学习-admin管理工具
date: 2019-12-20 09:08:58
tags: [python,django]
---

# Django Admin 管理工具

Django 提供了基于 web 的管理工具。

Django 自动管理工具是 django.contrib 的一部分。你可以在项目的 settings.py 中的 INSTALLED_APPS 看到它：

```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
)
```

django.contrib是一套庞大的功能集，它是Django基本代码的组成部分。

<!--more-->

## 激活管理工具

```python
默认在urls.py 打开

urlpatterns = [
    # 路径名称 ,路径方法,路径名称 用来反向显示
    path('admin/', admin.site.urls),
 ]
```

## 使用管理工具

启动开发服务器，然后在浏览器中访问 http://127.0.0.1:8000/admin/

你可以通过命令 **python manage.py createsuperuser** 来创建超级用户，如下所示：

```python
# python manage.py createsuperuser
Username (leave blank to use 'root'): admin
Email address: admin@runoob.com
Password:
Password (again):
Superuser created successfully.
[root@solar HelloWorld]#
```

为了让 admin 界面管理某个数据模型，我们需要先注册该数据模型到 admin。比如，我们之前在 TestModel 中已经创建了模型 Test 。修改 TestModel/admin.py:

```
from django.contrib import admin
from TestModel.models import Test
 
# Register your models here.
admin.site.register(Test)
```

刷新后即可看到 Testmodel 数据表

