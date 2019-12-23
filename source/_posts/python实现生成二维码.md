---
title: python实现生成二维码
date: 2019-12-23 16:03:27
tags: [python,django]
---

# 首先要导入需要的包

```
conda install -c conda-forge qrcode
python图像处理包
conda install -c conda-forge pillow
```

## 实现很简单

### python版本

```python
# -*- coding: utf-8 -*-
import qrcode
from PIL import Image, ImageDraw


def showQrCode():
    img = qrcode.make('http://www.linjingc.top')
    # img <qrcode.image.pil.PilImage object at 0x1044ed9d0>

    with open('test.png', 'wb') as f:
        img.save(f)


if __name__ == '__main__':
    showQrCode()



```

<!--more-->

### django版本

直接返回图片流输出页面

```python
# -*- coding: utf-8 -*-
from django.http import HttpResponse
import qrcode
from django.utils.six import BytesIO

'''
django具体实现类
传输data 图片流 返回html页面
'''

def generate_qrcode(request, data):
    img = qrcode.make(data)

    buf = BytesIO()
    img.save(buf)
    image_stream = buf.getvalue()

    response = HttpResponse(image_stream, content_type="image/png")
    return response
```

urls.py

```python
from django.contrib import admin
from django.urls import path

from qrCodeTest import qrController

urlpatterns = [
    path('qrcode/<str:data>/', qrController.generate_qrcode),
]

```

