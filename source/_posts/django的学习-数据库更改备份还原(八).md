---
title: django的学习-数据库更改备份还原(八)
date: 2019-12-20 11:56:05
tags: [python,django]
---

## Django 1.7.x 和后来的版本：

## Django 1.7.x 及以后的版本集成了 South 的功能，**在修改models.py了后运行：**

```
python manage.py makemigrations
python manage.py migrate
```

这两行命令就会对我们的models.py 进行检测，自动发现需要更改的，应用到数据库中去。

## Django 1.6.x 及以前：

写过Django项目的同学，必然会遇到这个问题：

在Django 1.6以及以前的版本中，我们测试，当发现model要改，怎么办？

<!--more-->



我们修改了 models.py 之后，我们运行：

```
python manage.py syncdb
```

这句话只会将我们在 models.py 中新加的类创建相应的表。

对于原来有的，现在删除了的类，Django 会询问是否要删除数据库中已经存在的相关数据表。



如果在原来的类上增加字段或者删除字段，可以参考这个命令：

```
python manage.py sql appname
```

给出的SQL语句，然后自己手动到数据库执行 SQL 。但是这样非常容易出错！

Django 的第三方 app South 就是专门做数据库表结构自动迁移工作，Jacob Kaplan-Moss 曾做过一次调查，South 名列最受欢迎的第三方 app。事实上，它现在已经俨然成为 Django 事实上的数据库表迁移标准，很多第三方 app 都会带 South migrations 脚本，Django 1.7 中集成了 South 的功能。



## **1, 安装South**

```
(sudo) pip install South
```



## **2. 使用方法**

一个好的程序使用起来必定是简单的，South和它的宗旨一样，使用简单。只需要简单几步，针对已经建好model和创建完表的应用。



把south加入到settings.py中的INSTALL_APPS中

```
# Application definition
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
 
    'blog',
    'south',
)
```

修改好后运行一次 python manage.py syncdb，Django会新建一个 south_migrationhistory 表，用来记录数据表更改(Migration)的历史纪录。

```
$ python manage.py syncdb
Syncing...
Creating tables ...
Creating table south_migrationhistory
Installing custom SQL ...
Installing indexes ...
No fixtures found.
 
Synced:
 > django.contrib.admin
 > django.contrib.auth
 > django.contrib.contenttypes
 > django.contrib.sessions
 > django.contrib.messages
 > django.contrib.staticfiles
 > blog
 > south
 
Not synced (use migrations):
```



如果要把之前建好的比如 blog 这个 app 使用 South 来管理：

```
$ python manage.py convert_to_south blog
```

你会发现blog文件夹中多了一个 migrations 目录，里面有一个 0001_initial.py 文件。



注：如果 blog 这个 app 之前就创建过相关的表，可以用下面的来“假装”用 South 创建（伪创建，在改动 models.py 之前运行这个）

```
python manage.py migrate blog --fake
```

意思是这个表我以前已经建好了，用 South 只是纪一下这个创建记录，下次 migrate 的时候不必再创建了。

原理就是 south_migrationhistory 中记录下了 models.py 的修改的历史，下次再修改时会和最近一次记录比较，发现改变了什么，然后生成相应的对应文件，最终执行相应的 SQL 更改原有的数据表。



接着，当你对 Blog.models 做任何修改后，只要执行：

```
$ python manage.py schemamigration blog --auto
```

South就会帮助我们找出哪些地方做了修改，如果你新增的数据表没有给default值，并且没有设置null=True, south会问你一些问题，因为新增的column对于原来的旧的数据不能为Null的话就得有一个值。顺利的话，在migrations文件夹下会产生一个0002_add_mobile_column.py，但是这一步并没有真正修改数据库的表，我们需要执行 python manage.py migrate ：

```

$ python manage.py migrate
Running migrations for blog:
 - Migrating forwards to 0002_add_mobile_column.
 > blog:0002_add_mobile_column
 - Loading initial data for blog.
No fixtures found.
```

这样所做的更改就写入到了数据库中了。



**恢复到以前**

South好处就是可以随时恢复到之前的一个版本，比如我们想要回到最开始的那个版本：

```
> python manage.py migrate blog 0001
 - Soft matched migration 0001 to 0001_initial.
Running migrations for blog:
 - Migrating backwards to just after 0001_initial.
 < blog:0002_add_mobile_column
```

这样就搞定了，数据库就恢复到以前了，比你手动更改要方便太多了。