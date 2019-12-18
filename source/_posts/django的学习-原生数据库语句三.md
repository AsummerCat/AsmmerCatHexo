---
title: django的学习-原生数据库语句三
date: 2019-12-18 15:10:01
tags: [python,django]
---

```python
# -*- coding: utf-8 -*-
# 原生sql
from django.http import HttpResponse
from django.db import connection


def index(request):
    cursor = connection.cursor()
    # 要想使用sql原生语句，必须用到execute()函数
    # 插入操作
    cursor.execute("insert into hello_author(name) values('郭敬明')")
    # 更新操作
    cursor.execute("update hello_author set name='abc ' where name='bcd'")
    # 删除操作
    cursor.execute("delete from hello_author where name='abc'")
    # 查询操作
    cursor.execute('select * from hello_author')
    raw = cursor.fetchone()  # 返回一条数据
    rows = cursor.fetchall()  # 返回查询到的所有数据
    for r in rows:
        print(r)

    return HttpResponse("输出测试数据")

```

