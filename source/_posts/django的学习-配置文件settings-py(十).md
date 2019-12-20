---
title: django的学习-配置文件settings.py(十)
date: 2019-12-20 15:49:09
tags: [python,django]
---

# settings.py

## 文件路径

```
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))
```

这里用到了python中一个神奇的变量 __file__ 这个变量可以获取到当前文件（包含这个代码的文件）的路径。os.path.dirname(__file__) 得到文件所在目录，再来一个os.path.dirname()就是目录的上一级，BASE_DIR 即为 项目 所在目录。我们在后面的与目录有关的变量都用它，这样使得移植性更强。

## DEBUG开关

```
# SECURITY WARNING: don't run with debug turned on in production!
DEBUG ``=` `True
TEMPLATE_DEBUG ``=` `True
```

**DEBUG＝True** 时，如果出现 bug 便于我们看见问题所在，但是部署时最好不要让用户看见bug的详情，可能一些不怀好心的人攻击网站，造成不必要的麻烦。



<!--more-->

## 是否ip或者域名允许访问

```
ALLOWED_HOSTS = ['*.besttome.com','www.ziqiangxuetang.com']
```

ALLOWED_HOSTS 允许你设置哪些域名可以访问，即使在 Apache 或 Nginx 等中绑定了，这里不允许的话，也是不能访问的。

当 **DEBUG=False** 时，这个为必填项，如果不想输入，可以用 **ALLOW_HOSTS = ['\*']** 来允许所有的。

## 静态文件处理

static 是静态文件所有目录，比如 jquery.js, bootstrap.min.css 等文件。

一般来说我们只要把静态文件放在 APP 中的 static 目录下，部署时用 python manage.py collectstatic 就可以把静态文件收集到（复制到） STATIC_ROOT 目录，但是有时我们有一些共用的静态文件，这时候可以设置 **STATICFILES_DIRS** 另外弄一个文件夹，如下：

```
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "common_static"),
    '/var/www/static/',
)
```

这样我们就可以把静态文件放在 common_static 和 /var/www/static/中了，Django也能找到它们。

## 用户存放文件

```
MEDIA_URL ``=` `'/media/'
MEDIA_ROOT ``=` `os.path.join(BASE_DIR,``'media'``)
```

## html模板存放位置

有时候有一些模板不是属于app的，比如 baidutongji.html, share.html等，

Django 1.5 - Django 1.7

```
TEMPLATE_DIRS = (
    os.path.join(BASE_DIR, 'templates').replace('\\', '/'),
    os.path.join(BASE_DIR, 'templates2').replace('\\', '/'),
    # ...
)
```

Django 1.8 及以上版本

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [
            os.path.join(BASE_DIR,'templates').replace('\\', '/'),
            os.path.join(BASE_DIR,'templates2').replace('\\', '/'),
        ],
        'APP_DIRS': True,
]
```

这样 就可以把模板文件放在 templates 和 templates2 文件夹中了。

