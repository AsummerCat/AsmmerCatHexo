---
title: python使用aria2下载
date: 2019-12-31 15:42:56
tags: [python,aria2]
---

# python使用aria2下载

## 导入相关模块

```python
import os
import time
from pyaria2 import Aria2RPC

'''
pip install pyaria2
'''
```

## 根据文件链接+文件名称 添加下载任务

```python
def get_file_from_url(link, file_name):
    jsonrpc = Aria2RPC()
    set_dir = os.path.dirname(__file__)
    options = {"dir": set_dir, "out": file_name, }
    res = jsonrpc.addUri([link], options=options)
```

## 根据文件链接 添加下载任务

```python
# 根据文件链接 添加下载任务
def get_file_from_cmd(link):
    exe_path = r'E:\aria2-1.35.0\aria2c.exe --conf-path=config.conf'
    order = exe_path + ' -s16 -x10 ' + link
    os.system(order)
```

<!--more-->

```python
# -*- coding:utf-8 -*-
import os
import time
from pyaria2 import Aria2RPC

'''
pip install pyaria2
'''


# 根据文件链接+文件名称 添加下载任务
def get_file_from_url(link, file_name):
    jsonrpc = Aria2RPC()
    set_dir = os.path.dirname(__file__)
    options = {"dir": set_dir, "out": file_name, }
    res = jsonrpc.addUri([link], options=options)

# 根据文件链接 添加下载任务
def get_file_from_cmd(link):
    exe_path = r'E:\aria2-1.35.0\aria2c.exe --conf-path=config.conf'
    order = exe_path + ' -s16 -x10 ' + link
    os.system(order)


if __name__ == '__main__':
    link = 'http://music.163.com/song/media/outer/url?id=400162138.mp3'
    filename = '海阔天空.mp3'

    start = time.time()
    get_file_from_url(link, filename)
    end = time.time()
    print(f"耗时:{end - start:.2f}")

```

# 控制台

```
http://aria2c.com/
```

