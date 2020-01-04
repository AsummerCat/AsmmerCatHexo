---
title: python-urllib库的使用
date: 2020-01-04 16:49:13
tags: [python]
---

# python-urllib库的使用

## 模块导入

```python
from urllib import request
# 生成POST请求数据
from urllib import parse
```

## 创建GET请求

```python
req=request.Request("http://www.baidu.com")
req.add_header("User-Agent","模拟用户浏览器")
resp=request.urlopen(req)
# 返回输出
print(resp.read().decode("utf-8"))
```

```python
resp=request.urlopen("http://www.baidu.com")
# 返回输出
print(resp.read().decode("utf-8"))
```



<!--more-->

## 创建POST请求

```python
# 构造post数据
postData =parse.urlencode([
(key1,val1),
(key2,val2)
])
req=request.Request("http://www.baidu.com")
req.add_header("User-Agent","模拟用户浏览器")
resp=request.urlopen(req,data=postData.encode('utf-8'))
# 返回输出
print(resp.read().decode("utf-8"))

```

## 添加消息头

```python
req.add_header("User-Agent","模拟用户浏览器")
req.add_header("User-Agent","模拟用户浏览器")
可以添加多个消息头
```



