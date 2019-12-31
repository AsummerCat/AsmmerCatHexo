---
title: python爬虫-关于解决scrapy请求http500丶502的问题
date: 2019-12-31 10:22:00
tags: [python,爬虫,scrapy]
---

# 关于解决scrapy请求http 500 502的问题

http 500 502是内部服务器错误，这个大家都晓得。
但有的网站在针对爬虫访问的时候也会利用错误码500或502来反扒

## 大致分为以下两种情况

### **第一次给出500或502的错误码，然后给出200的正常返回**

这样的情况很好处理，只要遇到这两个错误码就重新请求就好了。
 如果错误500，scrapy会自动重新请求，但502貌似不会，这时候只要在setting里面修改一下设置

```
RETRY_HTTP_CODES = [500, 502, 503, 504, 400, 403, 404, 408]
```

这个设置项的意思是遇到这些错误码就重新发送请求，但是如果错误码不在这里就不会重新请求，所以一定要填写所有需要重新请求的情况。如果想要遇到错误就忽略掉，从来都不重新请求，就把它设成等于`[]`就好了。

<!--more-->

### 修改scrapy下载器中间件，在setting里面做如下设置

```python
DOWNLOADER_MIDDLEWARES = {
'scrapy.contrib.downloadermiddleware.httpproxy.HttpProxyMiddleware': 110,   
'cnca.middlewares.ProxyMiddleware': 100,}
```

然后在setting同级目录下创建middlewares.py文件

```python
class ProxyMiddleware(object):
    def process_response(self, request, response, spider):
        if response.status == 502:      
            return response
```

