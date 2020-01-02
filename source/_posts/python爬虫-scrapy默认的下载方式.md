---
title: python爬虫-scrapy默认的下载方式
date: 2020-01-02 11:44:35
tags: [python,爬虫,scrapy]
---

# python爬虫-scrapy默认的下载方式

[demo地址](https://github.com/AsummerCat/scrapy_dongman_demo)

scrapy自带了file,image,media三个Pipeline

## 下载测试

###  导入模块

在pipelines模块中导入

```python
from scrapy.pipelines.images import ImagesPipeline  图片下载
from scrapy.pipelines.files import FilesPipeline    文件下载
from scrapy.pipelines.media import MediaPipeline    多媒体下载

```

<!--more-->

### 在配置文件中启用下载

添加对应的下载模块

```
ITEM_PIPELINE={'scrapy.pipelines.files.FilesPipeline':1,}
```

### 在items模块中定义两个关键字段

```python
class ExamplesItem(scrapy.Item):
    file_urls=scrapy.Field()
    files=scrapy.Field()
```

在siper中,只需要把要下载的URL添加给Item实例即可:

```python
 def parse(self,response):
        download_url=response.css('a::attr(href)').extract_first()#解析得到要下载的url
        item=ExamplesItem()
        item['file_urls']=[download_url]
        yield item
```

启动即可



# 自定义图片下载

## 在settings.py中设置下载路径

```python
# # 定义图片的保存路径
IMAGES_STORE = 'E:\动漫图'
```

## 在pipelines.py中添加

```python
from scrapy.pipelines.images import ImagesPipeline

class ImagesspiderPipeline(ImagesPipeline):

    def get_media_requests(self, item, info):
        '''获取图片的url,通过Request方法，保存图片'''
        # 这里meta={'item': item},目的事件item传递到file_path中
        return Request(item['down_path'][0], meta={'item': item})

    def file_path(self, request, response=None, info=None):
        '''图片保存的路径'''
        item = request.meta['item']
        return item["info_down_path"]

```

### 在settings.py中开启自定义下载

```python
ITEM_PIPELINES = {
    'scrapy_dongman_demo.pipelines.ScrapyDongmanDemoPipeline': 200,
    # 下载图片
    'scrapy_dongman_demo.pipelines.ImagesspiderPipeline': 300,
}
```

启动即可 下载