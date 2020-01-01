---
title: python使用aria2下载
date: 2019-12-31 15:42:56
tags: [python,aria2]
---

# python使用aria2下载

windows下使用前这种

在mac上使用后一种方式

* 官方文档:https://aria2.github.io/manual/en/html/aria2c.html#aria2.addUri



## 测试调用

## json

```h
请求: http://127.0.0.1:6800/jsonrpc

参数:  {"jsonrpc": "2.0", "id": "qwer",
 "method": "aria2.addUri",
 "params": [["https://vid3-l3.xvideos-cdn.com/videos/mp4/0/2/a/xvideos.com_02a33f4c76ec22df0ddefd1b35b2058a.mp4?e=1577905289&ri=1024&rs=85&h=a57452223c3a35de1c015d1955ad745a"],{"out":"1.mp4" ,"dir":"/Users/cat/Downloads/"}]
}
```



# 普通模式 使用xml   windows系统下

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

# Json模式   mac系统下

## 导入相关模块

```
import requests, json
```

## 下载

```python
def get_file_from_url_by_mac(path, link, file_name):
    options = {"dir": path, "out": file_name}
    params = [[link], options]
    url = "http://localhost:6800/jsonrpc"
    jsonreq = json.dumps({'jsonrpc': '2.0', 'id': 'qwer',
                          'method': 'aria2.addUri',
                          "params": params})
    requests.post(url, jsonreq)
```



# 通用方法

判断哪个操作系统 然后分别调用

```
# 检查当前系统
def check_os():
    sysstr = platform.system()

    if (sysstr == "Windows"):
        return "Windows"
    elif (sysstr == "Linux"):
        return "linux"
    elif (sysstr == "Darwin"):
        return "macOS"
    else:
        return "other"

```



# 控制台

```
http://aria2c.com/
```

