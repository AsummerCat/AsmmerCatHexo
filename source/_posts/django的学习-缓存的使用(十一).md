---
title: django的学习-缓存的使用(十一)
date: 2019-12-20 16:19:28
tags: [python,django]
---

# 缓存的使用

Django 官方关于cache的介绍：https://docs.djangoproject.com/en/dev/topics/cache/

Django 是动态网站，一般来说需要实时地生成访问的网页，展示给访问者，这样，内容可以随时变化，但是从数据库读多次把所需要的数据取出来，要比从内存或者硬盘等一次读出来 付出的成本大很多。

**缓存系统工作原理：**

对于给定的网址，尝试从缓存中找到网址，如果页面在缓存中，直接返回缓存的页面，如果缓存中没有，一系列操作（比如查数据库）后，保存生成的页面内容到缓存系统以供下一次使用，然后返回生成的页面内容。

<!--more-->

## **Django settings 中 cache 默认为**

```
{
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
    }
}
```

也就是默认利用**本地的内存**来当缓存，速度很快。当然可能出来内存不够用的情况，其它的一些内建可用的 Backends 有

```

'django.core.cache.backends.db.DatabaseCache'
'django.core.cache.backends.dummy.DummyCache'
'django.core.cache.backends.filebased.FileBasedCache'
'django.core.cache.backends.locmem.LocMemCache'
'django.core.cache.backends.memcached.MemcachedCache'
'django.core.cache.backends.memcached.PyLibMCCache'
```

在 github 上也有用 redis 做 Django的缓存系统的开源项目：https://github.com/niwibe/django-redis

## 利用文件系统来缓存

```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': '/var/tmp/django_cache',
        'TIMEOUT': 600,
        'OPTIONS': {
            'MAX_ENTRIES': 1000
        }
    }
}
```

利用数据库来缓存，利用命令创建相应的表：python manage.py createcachetable cache_table_name

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'cache_table_name',
        'TIMEOUT': 600,
        'OPTIONS': {
            'MAX_ENTRIES': 2000
        }
    }
}
```

# 实例

下面用一些实例来说明如何使用 Django 缓存系统

**一般来说我们用 Django 来搭建一个网站，要用到数据库等**

```
from django.shortcuts import render
def index(request):
    # 读取数据库等 并渲染到网页
    # 数据库获取的结果保存到 queryset 中
    return render(request, 'index.html', {'queryset':queryset})
```

像这样每次访问都要读取数据库，一般的小网站没什么问题，当访问量非常大的时候，就会有很多次的数据库查询，肯定会造成访问速度变慢，服务器资源占用较多等问题。

```
from django.shortcuts import render
from django.views.decorators.cache import cache_page
 
@cache_page(60 * 15) # 秒数，这里指缓存 15 分钟，不直接写900是为了提高可读性
def index(request):
    # 读取数据库等 并渲染到网页
    return render(request, 'index.html', {'queryset':queryset})
```

当使用了cache后，访问情况变成了如下：

```
访问一个网址时, 尝试从 cache 中找有没有缓存内容
如果网页在缓存中显示缓存内容，否则生成访问的页面，保存在缓存中以便下次使用，显示缓存的页面。
given a URL, try finding that page in the cache
if the page is in the cache:
    return the cached page
else:
    generate the page
    save the generated page in the cache (for next time)
    return the generated page
```

**Memcached 是目前 Django 可用的最快的缓存**

***\*另外，Django 还可以共享缓存。\****