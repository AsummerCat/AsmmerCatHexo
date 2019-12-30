---
title: python爬虫-scrapy伪装方式
date: 2019-12-30 22:22:08
tags: [python,爬虫,scrapy]
---

# python爬虫-scrapy伪装方式

防止抓取失败

或者被封ip

## 代理ip

<!--more-->

### 在middlewares.py中编写代理ip的方法

```python
'''
ip代理
https://www.kuaidaili.com/free/intr/
https://www.xicidaili.com/nn/
'''

class my_proxy(object):
    def process_request(self, request, spider):
        # 代理服务器
        request.meta['proxy'] = 'http-cla.abuyun.com:9030'
        # 代理服务器用户名密码 :分隔
        proxy_name_pass = b'H211EATS905745KC:F8FFBC929EB7D5A7'
        # 用户名密码加密
        encode_pass_name = base64.b64encode(proxy_name_pass)
        # 设置http头
        request.headers['Proxy-Authorization'] = 'Basic ' + encode_pass_name.decode()

```

需要添加协议名称，`http://或者https://`

```python
class my_proxy(object):
    PROXIES = ['110.52.235.131:9999','110.52.235.249:9999','112.17.38.141:3128']
 
    def process_request(self,request,spider):
        proxy = random.choice(self.PROXIES)
        request.meta['proxy'] = 'http://'+proxy
```

### 在setting.py中开启代理服务器

```python

DOWNLOADER_MIDDLEWARES = {
   # 'scrapy_dongman_demo.middlewares.ScrapyDongmanDemoDownloaderMiddleware': 543,
   # 开启ip代理
   'scrapy_dongman_demo.middlewares.my_proxy': 544,
}
```

如果没报错 表示通过





## 伪装USER-AGENT

### 在middlewares.py中编写随机设置USER-AGENT的方法

```python
'''
随机user_agent
https://www.jianshu.com/p/8e115373b101
'''


class my_user_agent(object):
    pcUserAgent = [
        # safari 5.1 – Windows
        "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
        # Firefox 38esr
        "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:38.0) Gecko/20100101 Firefox/38.0",
        # Opera 11.11 – Windows
        "Opera/9.80 (Windows NT 6.1; U; en) Presto/2.8.131 Version/11.11"
    ]

    def process_request(self, request, spider):
        request.headers['User_Agent'] = random.choice(self.pcUserAgent)
```

### 在setting.py中开启随机USER_AGENT

```python
DOWNLOADER_MIDDLEWARES = {
   # 'scrapy_dongman_demo.middlewares.ScrapyDongmanDemoDownloaderMiddleware': 543,
    # 开启代理
   'scrapy_dongman_demo.middlewares.my_proxy': 544,
    #伪装user_agent
   'scrapy_dongman_demo.middlewares.my_user_agent': 545,
}
```

