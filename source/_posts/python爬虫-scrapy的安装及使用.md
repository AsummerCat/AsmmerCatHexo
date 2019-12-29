---
title: python爬虫-scrapy的安装及使用
date: 2019-12-26 21:23:43
tags: [python,爬虫,scrapy]
---

# scrapy

## 安装

### python

```
pip install scrapy
```

### anaconda

```
conda install scrapy
```



<!--more-->

# 这边会用都xPath解析页面

## 下载Chrome插件

```
XPath Helper 
```

或者

火狐的

```
try xpath 
```



# 创建项目

```
scrapy startproject xxx
```

## 构建爬虫

```
cd 进入spiders目录 然后
前面是构建名称 最后一个是抓取的域名
 scrapy genspider douban douban.com  
```



## 启动爬虫

进入spider目录

```
scrapy crawl XXX   xxx表示爬虫名称
```

## 导出爬取的数据

```
scrapy crawl XXX -o  xxx.json
或者
scrapy crawl XXX -o  xxx.csv
```



# 业务逻辑

## spider.py

```python
# -*- coding: utf-8 -*-
import scrapy

'''
爬虫文件

'''


class DoubanSpidersSpider(scrapy.Spider):
    # 爬虫名称
    name = 'douban_spiders'
    # 允许的域名 爬虫
    allowed_domains = ['movie.douban.com']
    # 入口url,扔到调度器
    start_urls = ['http://movie.douban.com/top250']

    def parse(self, response):
        print("请求成功")


```

## item.py

```python
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://doc.scrapy.org/en/latest/topics/items.html

import scrapy


class ScrapyDemoItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    # # 需要抓取的逻辑
    movie_num = scrapy.Field()  # 序号
    introduce = scrapy.Field()  # 电影介绍
    star = scrapy.Field()  # 星级
    evaluate = scrapy.Field()  # 评论
    describe = scrapy.Field()  # 描述
    pass

```