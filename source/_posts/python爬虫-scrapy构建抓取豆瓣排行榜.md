---
title: python爬虫-scrapy构建抓取豆瓣排行榜
date: 2019-12-29 22:08:11
tags: [python,爬虫,scrapy]
---

# 创建项目

```
scrapy startproject start douban_demo
```



## 构建爬虫

```
scrapy genspider douban_spiders  movie.douban.com 
```



## 启动爬虫

进入spider目录

```
scrapy crawl douban_spiders   
```

<!--more-->

## 导出爬取的数据

```python
scrapy crawl douban_spiders -o  test.json
或者
scrapy crawl douban_spiders -o  test.csv
```



# 代码编写

## settngs.py 设置

```python
开启用户代理
USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36'

延迟请求时间 可以不设置
DOWNLOAD_DELAY = 3
```

## 主入口创建

mian.py

```python
# -*- coding: utf-8 -*-
from scrapy import cmdline
'''
自定义主入口
启动脚本
'''

if __name__ == '__main__':
    cmdline.execute('scrapy crawl douban_spiders'.split())
```



