---

title: python爬虫-scrapy抓取动漫图自定义图片下载
date: 2020-01-02 12:01:11
tags: [python,爬虫,scrapy]
---

#  python爬虫-scrapy抓取动漫图自定义图片下载

## [demo 地址](https://github.com/AsummerCat/scrapy_dongman_demo)

## 设置items.py 定义结构

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


class ScrapyDongmanDemoItem(scrapy.Item):
    # define the fields for your item here like:
    # 套图标题
    title = scrapy.Field()
    # 套图详情页标题
    detail_title = scrapy.Field()
    # 套图地址
    url = scrapy.Field()
    # 详细页地址
    pic_url = scrapy.Field()
    # 详细页图片下载地址
    down_path = scrapy.Field()
    # 下载电脑的地址
    info_down_path = scrapy.Field()

```

<!--more-->

## 编写爬虫类

dongman_spiders.py

```python
# -*- coding: utf-8 -*-
import re

import scrapy
from scrapy_dongman_demo.items import ScrapyDongmanDemoItem

'''
抓取 图片爬虫

'''

# 全局页数
page_num = 0


class DongmanSpidersSpider(scrapy.Spider):
    name = 'dongman_spiders'
    allowed_domains = ['www.mmonly.cc']
    start_urls = ['http://www.mmonly.cc/ktmh']

    def parse(self, response):
        pic_index_list = response.xpath(
            "//div[@class='Clbc_Game_l_a']//div[@id='infinite_scroll']/div[@class='item masonry_brick masonry-brick']/div[@class='item_t']/div/div[@class='ABox']/a")
        for i in pic_index_list:
            # 遍历节点
            data = ScrapyDongmanDemoItem()
            data['title'] = i.xpath("./img/@alt").extract_first()
            data['url'] = i.xpath("./@href").extract_first()
            data['pic_url'] = i.xpath("./@href").extract_first()
            yield scrapy.Request(
                data['url'], callback=self.parse_detail, meta={"item": data})
        # # 下一页
        # next_url = response.xpath("//div[@id='pageNum']/a[last()-1]/@href").extract()
        # try:
        #     if next_url:
        #         next_link = next_url[0]
        #         # 提交管道进行 下一页处理
        #         yield scrapy.Request("".join(self.start_urls) + "/" + next_link, callback=self.parse)
        # except:
        #     pass

    '''
    获取详细页数据 
    '''

    def parse_detail(self, response):
        item = response.meta["item"]
        xpath_html = response.xpath("//div[@class='photo']/div[@class='wrapper clearfix imgtitle']")
        # 名称前缀
        detail_title_prefix = xpath_html.xpath("./h1/text()").extract()
        # 名称后缀
        detail_title_suffix = xpath_html.xpath("string(./h1/span)").extract()
        # 明细页名称
        item["detail_title"] = "".join(detail_title_prefix + detail_title_suffix)
        # 详细页图片下载地址
        item["down_path"] = xpath_html.xpath("./ul/li[@class='pic-down h-pic-down']/a/@href").extract()
        yield item
        next_url = response.xpath("//div[@class='pages']/ul/li[@id='nl']/a/@href").extract()
        try:
            if next_url and not '##' == next_url[0]:
                next_link = next_url[0]
                # 提交管道进行 下一页处理
                pic_url = item['pic_url']
                next_page_flag = pic_url[pic_url.rfind('/', 1) + 1:len(pic_url) + 1]

                data = ScrapyDongmanDemoItem()
                data['title'] = item['title']
                data['url'] = item['url']
                data['pic_url'] = str(item['pic_url']).replace(next_page_flag, next_link)
                yield scrapy.Request(data['pic_url'], callback=self.parse_detail, meta={"item": data})
        except:
            pass
```

输出内容给管道

## 定义管道

### 这边有两个类

一个类是获取item数据处理

一个是获取item下载图片

```python
# -*- coding: utf-8 -*-

# Define your item pipelines here
#
# Don't forget to add your pipeline to the ITEM_PIPELINES setting
# See: https://doc.scrapy.org/en/latest/topics/item-pipeline.html


'''
管道数据处理
两个管道 一个管道输出item传递给下一个
'''
import os
import re
from scrapy import Request

from scrapy.pipelines.images import ImagesPipeline


class ScrapyDongmanDemoPipeline(object):
    def process_item(self, item, spider):
        title = "".join(re.findall('[\u4e00-\u9fa5a-zA-Z0-9]+', item['title'], re.S))
        path = 'E:\\动漫图\\{}'.format(title)
        # print("开始下载动漫图:{},详细页:{}]".format(title, item['detail_title']))
        # 判断文件夹是否存在 不存在直接makedirs 创建多级目录
        if not os.path.exists(path):
            os.makedirs(path)
            # 获取下载的文件名称
        item["info_down_path"] = path + "\\" + "".join(
            re.findall('[\u4e00-\u9fa5a-zA-Z0-9]+',
                       str(item['detail_title']).format("(", "第").replace("/", "分").replace(")", "页"), re.S)) + ".jpg"
        return item


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



## 在middlewares.py中添加伪装User_agent

```python

class my_user_agent(object):
    pcUserAgent = [
        "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
        "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:38.0) Gecko/20100101 Firefox/38.0",
        "Opera/9.80 (Windows NT 6.1; U; en) Presto/2.8.131 Version/11.11"
    ]

    def process_request(self, request, spider):
        request.headers['User_Agent'] = random.choice(self.pcUserAgent)
```

## 最后在settings.py中开启对应的配置

```
# 是否根据爬虫协议抓取
ROBOTSTXT_OBEY = False
# 下载中间件
DOWNLOADER_MIDDLEWARES = {
    #伪装user_agent
   'scrapy_dongman_demo.middlewares.my_user_agent': 545,
}
# item管道
ITEM_PIPELINES = {
    'scrapy_dongman_demo.pipelines.ScrapyDongmanDemoPipeline': 200,
    # 下载图片
    'scrapy_dongman_demo.pipelines.ImagesspiderPipeline': 300,
}

# # 定义图片的保存路径
IMAGES_STORE = 'E:\动漫图'

# 修改编码为utf-8
FEED_EXPORT_ENCODING = 'utf-8'

```

## 最最后设置一个启动类

mian.py

```python
# -*- coding: utf-8 -*-
from scrapy import cmdline

'''
自定义主入口
启动脚本
'''

if __name__ == '__main__':
    cmdline.execute('scrapy crawl dongman_spiders'.split())

```

