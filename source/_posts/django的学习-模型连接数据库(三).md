---
title: django的学习-模型连接数据库(三)
date: 2019-12-17 15:28:19
tags: [python,django]
---

## [demo地址](https://github.com/AsummerCat/gogogo/blob/0f571b4db3e648b2edfd6a5eb786ed6931c4b753/gogogo/dbTest.py)

Django 对各种数据库提供了很好的支持，包括：PostgreSQL、MySQL、SQLite、Oracle。

Django 为这些数据库提供了统一的调用API。 我们可以根据自己业务需求选择不同的数据库。

MySQL 是 Web 应用中最常用的数据库。本章节我们将以 Mysql 作为实例进行介绍。你可以通过本站的[ MySQL 教程](https://www.runoob.com/django/mysql/mysql-tutorial.html) 了解更多Mysql的基础知识。

如果你没安装 mysql 驱动，可以执行以下命令安装：

```
sudo pip install mysqlclient
```

## (必须)数据库配置

我们在项目的 settings.py 文件中找到 DATABASES 配置项，将其信息修改为：

<!--more-->

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',  # 或者使用 mysql.connector.django
        'NAME': 'test',
        'USER': 'test',
        'PASSWORD': 'test123',
        'HOST':'localhost',
        'PORT':'3306',
    }
}
```

这里添加了中文注释，所以你需要在 HelloWorld/settings.py 文件头部添加 **# -\*- coding: UTF-8 -\*-**。

上面包含数据库名称和用户的信息，它们与 MySQL 中对应数据库和用户的设置相同。Django 根据这一设置，与 MySQL 中相应的数据库和用户连接起来。

## (必须)添加apps

```python
在settings.py中找到INSTALLED_APPS这一项，如下：
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'gogogo',               # 添加此项
)
```

这边表示添加项目到路径中

## 定义模型

我们修改 TestModel/models.py 文件，代码如下：

```python
# models.py
from django.db import models
 
class Test(models.Model):
    name = models.CharField(max_length=20)
```

以上的类名代表了数据库表名，且继承了models.Model，类里面的字段代表数据表中的字段(name)，数据类型则由CharField（相当于varchar）、DateField（相当于datetime）， max_length 参数限定长度。

接下来在settings.py中找到INSTALLED_APPS这一项，如下：

```python
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'TestModel',               # 添加此项
)
```

## 如果已经存在的数据库 可以进行导出实体构建

`pip install mysqlclient`

运行命令：

`python manage.py inspectdb > models.py`

然后，在 manage.py 的 下面就会出现一个 models.py，里面包含了已有数据库的所有表及结构。
内容如下：

```
class User(models.Model):
    name = models.CharField(max_length=255)
    password = models.CharField(max_length=255, blank=True, null=True)
    
    class Meta:
        managed = False
        db_table = 'user'
```

把想要使用的表的 managed 都改成 True。`managed = True`
运行命令：
`python manage.py migrate`
成功后就可以对已有数据库进行 ORM 操作了。

(注意:) 每张表必须都要有主键 不然可能查询失败

## 清空数据库

```python
python manage.py flush
```

此命令会询问是 yes 还是 no, 选择 yes 会把**数据全部清空掉**，只留下空表。

## 数据库操作 

## 导包处理

```python
from gogogo.models import *
```



## 新增

```python
from django.http import HttpResponse
 
from TestModel.models import Test
 
# 数据库操作
def testdb(request):
    test1 = Test(name='runoob')
    test1.save()
    return HttpResponse("<p>数据添加成功！</p>")
```

### 获取数据

```python
# -*- coding: utf-8 -*-
 
from django.http import HttpResponse
 
from TestModel.models import Test
 
# 数据库操作
def testdb(request):
    # 初始化
    response = ""
    response1 = ""
    
    
    # 通过objects这个模型管理器的all()获得所有数据行，相当于SQL中的SELECT * FROM
    list = Test.objects.all()
        
    # filter相当于SQL中的WHERE，可设置条件过滤结果
    response2 = Test.objects.filter(id=1) 
    
    # 获取单个对象
    response3 = Test.objects.get(id=1) 
    
    # 限制返回的数据 相当于 SQL 中的 OFFSET 0 LIMIT 2;
    Test.objects.order_by('name')[0:2]
    
    #数据排序
    Test.objects.order_by("id")
    
    # 上面的方法可以连锁使用
    Test.objects.filter(name="runoob").order_by("id")
    
    # 输出所有数据
    for var in list:
        response1 += var.name + " "
    response = response1
    return HttpResponse("<p>" + response + "</p>")
```

### 更新数据

修改数据可以使用 save() 或 update():

```python
# -*- coding: utf-8 -*-
 
from django.http import HttpResponse
 
from TestModel.models import Test
 
# 数据库操作
def testdb(request):
    # 修改其中一个id=1的name字段，再save，相当于SQL中的UPDATE
    test1 = Test.objects.get(id=1)
    test1.name = 'Google'
    test1.save()
    
    # 另外一种方式
    #Test.objects.filter(id=1).update(name='Google')
    
    # 修改所有的列
    # Test.objects.all().update(name='Google')
    
    return HttpResponse("<p>修改成功</p>")
```

### 删除数据

删除数据库中的对象只需调用该对象的delete()方法即可：

```python
# -*- coding: utf-8 -*-
 
from django.http import HttpResponse
 
from TestModel.models import Test
 
# 数据库操作
def testdb(request):
    # 删除id=1的数据
    test1 = Test.objects.get(id=1)
    test1.delete()
    
    # 另外一种方式
    # Test.objects.filter(id=1).delete()
    
    # 删除所有数据
    # Test.objects.all().delete()
    
    return HttpResponse("<p>删除成功</p>")
```

### 查看django queryset执行的sql

```
print str(Author.objects.all().query)
```

输出:

```
SELECT "blog_author"."id", "blog_author"."name", "blog_author"."qq", "blog_author"."addr", "blog_author"."email" FROM "blog_author"
```

### **以values_list 获取元组形式结果**

```python
authors = Author.objects.values_list('name', 'qq')
list(authors)


输出:
[(u'WeizhongTu', u'336643078'),

 (u'twz915', u'915792575'),

 (u'wangdachui', u'353506297'),

 (u'xiaoming', u'004466315')]

```

如果只需要 1 个字段，可以指定 flat=True

```
Author.objects.values_list('name', flat=True)
```

### **以values 获取字典形式的结果**

```
Author.objects.values('name', 'qq')
 list(Author.objects.values('name', 'qq'))
 
 输出:
 [{'name': u'WeizhongTu', 'qq': u'336643078'},

 {'name': u'twz915', 'qq': u'915792575'},

 {'name': u'wangdachui', 'qq': u'353506297'},

 {'name': u'xiaoming', 'qq': u'004466315'}]
```

### **extra 实现 别名，条件，排序等**

#### 别名

```
Tag.objects.all().extra(select={'tag_name': 'name'})

In [45]: tags[0].name
Out[45]: u'Django'
In [46]: tags[0].tag_name
Out[46]: u'Django'


```

### **defer 排除不需要的字段**

在复杂的情况下，表中可能有些字段内容非常多，取出来转化成 Python 对象会占用大量的资源。

这时候可以用 defer 来排除这些字段，比如我们在文章列表页，只需要文章的标题和作者，没有必要把文章的内容也获取出来（因为会转换成python对象，浪费内存）

```
原语句
Article.objects.all()
忽略content字段
Article.objects.all().defer('content')



```

### **only 仅选择需要的字段**

和 defer 相反，only 用于取出需要的字段，假如我们**只需要查出 作者的名称**

```python
Author.objects.all().only('name')
```

