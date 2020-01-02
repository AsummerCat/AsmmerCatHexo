---
title: python爬虫-scrapy抓取小视频批量检出调用aria2下载
date: 2020-01-02 13:48:35
tags: [python,爬虫,scrapy]
---

#  python爬虫-scrapy抓取小视频批量检出调用aria2下载

## 首先定义结构 items.py

```python
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://doc.scrapy.org/en/latest/topics/items.html

import scrapy

'''
结构定义

'''


class YellowDownloadItem(scrapy.Item):
    # define the fields for your item here like:
    # 标题
    title = scrapy.Field()
    # 列表地址
    url = scrapy.Field()
    # 视频下载地址
    down_path = scrapy.Field()
    # 下载电脑的地址
    info_down_path = scrapy.Field()
    # 下载本地目录
    file_path = scrapy.Field()

```

<!--more-->

# 下载中间件设置 伪装user_agent  middlewares.py

```python
class my_user_agent(object):
    pcUserAgent = [
        # safari 5.1 – Windows
        "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
        # Firefox 38esr
        "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:38.0) Gecko/20100101 Firefox/38.0",
        # Firefox 4.0.1 – MAC
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.6; rv:2.0.1) Gecko/20100101 Firefox/4.0.1",
        # Firefox 4.0.1 – Windows
        "Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/20100101 Firefox/4.0.1",
        # Opera 11.11 – MAC
        "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; en) Presto/2.8.131 Version/11.11",
        # safari 5.1 – MAC
        "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
        # Green Browser
        "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)",
        # Avant
        "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Avant Browser)",
        # 360浏览器
        "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; 360SE)",
        # 搜狗浏览器 1.x
        "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; SE 2.X MetaSr 1.0; SE 2.X MetaSr 1.0; .NET CLR 2.0.50727; SE 2.X MetaSr 1.0)",
        # Chrome 17.0 – MAC
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_0) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
        # Opera 11.11 – Windows
        "Opera/9.80 (Windows NT 6.1; U; en) Presto/2.8.131 Version/11.11"
    ]
    def process_request(self, request, spider):
        request.headers['User_Agent'] = random.choice(self.pcUserAgent)

```

### 开启settings.py的 下载中间件配置

```python
DOWNLOADER_MIDDLEWARES = {
# 伪装user_agent
   'yellow_download.middlewares.my_user_agent': 545,
}
```

## 编写爬虫类

```python
# -*- coding: utf-8 -*-
import scrapy

from yellow_download.items import YellowDownloadItem


class YellowSpidersSpider(scrapy.Spider):
    name = 'yellow_spiders'
    allowed_domains = ['mm006.xyz']
    # 搜索的内容
    key = '%E6%A0%A1%E8%8A%B1'
    # 详细页
    detail_url_prefix = "https://mm006.xyz{}"
    # 列表页
    url_prefix = "https://mm006.xyz/search.php?key={}&type=".format(key)

    start_urls = [url_prefix + '1']

    # 获取第一页数据及其所有列表
    def parse(self, response):
        index_list = response.xpath("//div[@class='item']")
        for i in index_list:
            # 遍历节点
            data = YellowDownloadItem()
            data['title'] = i.xpath(".//a[@class='movie-box']/div[@class='photo-info']/span/text()").extract()[0]
            data['url'] = i.xpath("./a[@class='movie-box']/@href").extract_first()
            yield scrapy.Request(self.detail_url_prefix.format(data['url']), callback=self.detail_pares,
                                 meta={"item": data})

            # 获取最大页数
        max_page = response.xpath("//div[@class='text-center']/ul/li/a[@class='end']/text()").extract_first()
        # 最大页数存在
        if max_page:
            for i in range(2, int(max_page) + 1):
                next_url = self.url_prefix + str(i)
                yield scrapy.Request(next_url, callback=self.next_parse)

    # 下一页
    def next_parse(self, response):
        index_list = response.xpath("//div[@class='item']")
        for i in index_list:
            # 遍历节点
            data = YellowDownloadItem()
            data['title'] = i.xpath(".//a[@class='movie-box']/div[@class='photo-info']/span/text()").extract()[0]
            data['url'] = i.xpath("./a[@class='movie-box']/@href").extract_first()
            yield scrapy.Request(self.detail_url_prefix.format(data['url']), callback=self.detail_pares,
                                 meta={"item": data})

    # 获取详情页的数据
    def detail_pares(self, response):
        data = response.meta["item"]
        my_video = response.xpath("//source")
        data["down_path"] = my_video[1].xpath("./@src").extract_first()
        yield data


```

输出data 进入管道

## 编写管道类 获取数据调用aria2下载

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html

'''
管道下载
'''
import json
import os
import platform
import re
import uuid

import requests
from pyaria2 import Aria2RPC
from yellow_download.settings import MAC_DOWNLOAD_PATH
from yellow_download.settings import WIN_DOWNLOAD_PATH


class YellowDownloadPipeline(object):
    def process_item(self, item, spider):
        title = "".join(re.findall('[\u4e00-\u9fa5a-zA-Z0-9]+', item['title'], re.S))
        path = ""
        info_os = check_os()
        if "Windows" == info_os:
            path = MAC_DOWNLOAD_PATH
        elif "macOS" == info_os:
            path = WIN_DOWNLOAD_PATH
        else:
            print("暂未识别系统无法下载内容")
        # 下载本地目录
        item["file_path"] = path
        # 判断文件夹是否存在 不存在直接makedirs 创建多级目录
        if not os.path.exists(path):
            os.makedirs(path)
        # 获取下载的文件名称
        if not len(title) > 0:
            title = str(uuid.uuid1()) + ".mp4"
        else:
            title = title + ".mp4"
        item["title"] = title
        info_down_path = path + "/" + title

        item["info_down_path"] = info_down_path
        if not os.path.exists(info_down_path):
            print("开始下载:{}".format(title))
            self.addTask(item=item)
        else:
            print(title + "====================>>>>已存在")
        return item

    '''
    添加下载任务
    '''

    def addTask(self, item):
        title = item["title"]
        down_path = item["down_path"]
        info_down_path = item["info_down_path"]
        path = item["file_path"]
        print("名称:{}=================下载地址:{}".format(title, info_down_path))
        self.get_file_from_url(path, down_path, title)

    # 根据文件链接+文件名称 添加下载任务
    def get_file_from_url(self, path, link, file_name):
        info_os = check_os()
        if "Windows" == info_os:
            get_file_from_url_by_windows(path, link, file_name)
        elif "macOS" == info_os:
            get_file_from_url_by_mac(path, link, file_name)
        else:
            print("暂未识别系统无法下载内容")


def get_file_from_url_by_windows(path, link, file_name):
    jsonrpc = Aria2RPC()
    options = {"dir": path, "out": file_name, }
    res = jsonrpc.addUri([link], options=options)


# json形式根据文件链接+文件名称 添加下载任务
def get_file_from_url_by_mac(path, link, file_name):
    options = {"dir": path, "out": file_name}
    params = [[link], options]
    url = "http://localhost:6800/jsonrpc"
    jsonreq = json.dumps({'jsonrpc': '2.0', 'id': 'qwer',
                          'method': 'aria2.addUri',
                          "params": params})
    requests.post(url, jsonreq)


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





## 开启settings.py的配置

```python


ROBOTSTXT_OBEY = False

DOWNLOADER_MIDDLEWARES = {
# 伪装user_agent
   'yellow_download.middlewares.my_user_agent': 545,
}

ITEM_PIPELINES = {
   'yellow_download.pipelines.YellowDownloadPipeline': 300,
}

# 修改编码为utf-8
FEED_EXPORT_ENCODING = 'utf-8'

# 捕获HttpCodes 重试请求
RETRY_HTTP_CODES = [500, 502, 503, 504, 400, 403, 404, 408]

# # 定义保存路径
MAC_DOWNLOAD_PATH = 'E:\测试下载'
WIN_DOWNLOAD_PATH = '/Users/cat/Downloads'

```

## 启动类

mian.py

```python
# -*- coding: utf-8 -*-
from scrapy import cmdline

'''
自定义主入口
启动脚本
'''

if __name__ == '__main__':
    cmdline.execute('scrapy crawl yellow_spiders'.split())

```

