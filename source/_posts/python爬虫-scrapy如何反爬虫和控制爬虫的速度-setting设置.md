---
title: python爬虫-scrapy如何反爬虫和控制爬虫的速度-setting设置
date: 2019-12-31 11:36:44
tags: [python,爬虫,scrapy]
---

# 如何反爬虫和控制爬虫的速度-setting设置

## 修改是否遵守爬虫协议为False

```python
# Obey robots.txt rules
ROBOTSTXT_OBEY = False
```

## 修改并发请求数，修改为1，或者2，越小爬取速度越慢，太快容易被识别到

```python
# Configure maximum concurrent requests performed by Scrapy (default: 16)
CONCURRENT_REQUESTS = 1
```

## 修改下载延迟时间，DOWNLOAD_DELAY设置越大请求越慢

```python
# Configure a delay for requests for the same website (default: 0)
# See https://doc.scrapy.org/en/latest/topics/settings.html#download-delay
# See also autothrottle settings and docs
DOWNLOAD_DELAY = 3
#默认False;为True表示启用AUTOTHROTTLE扩展
AUTOTHROTTLE_ENABLED = True
#默认5秒;初始下载延迟时间
AUTOTHROTTLE_START_DELAY = 1
#默认60秒；在高延迟情况下最大的下载延迟
AUTOTHROTTLE_MAX_DELAY = 3

```

<!--more-->

## 开启中间件

```python
# Enable or disable downloader middlewares
# See https://doc.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
   'askdoctor.middlewares.AskdoctorDownloaderMiddleware': 543,
}

```

## 开启PIPELINES，一般在要存储数据的时候开启

```python
# Configure item pipelines
# See https://doc.scrapy.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
   'askdoctor.pipelines.AskdoctorPipeline': 300,
}
```

## 开启如下设置

```python
# Enable and configure HTTP caching (disabled by default)
# See https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-middleware-settings
#开启本地缓存
HTTPCACHE_ENABLED = True
#将http缓存延迟时间
HTTPCACHE_EXPIRATION_SECS = 1
HTTPCACHE_DIR = 'httpcache'
HTTPCACHE_IGNORE_HTTP_CODES = []
HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'
```

爬取的过程中遇到一个问题就是，如果爬取页面设置为从page1到page10000,爬取的结果有很多漏掉的。然后将设置修改为如上，还是会有漏掉的。

 最后我的解决办法是将DOWNLOAD_DELAY 时间设置的更大一些。