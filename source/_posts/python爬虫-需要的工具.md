---
title: python爬虫-需要的工具
date: 2019-12-26 10:35:33
tags: [python,爬虫]
---

# python爬虫的准备知识

* python3.7

* Beautiful Soup库          ->抓取页面标签

 ```python
   soup = BeautifulSoup(sendHttp(url), 'html.parser')
      con = soup.find(id='position')
 ```

* requests库     ->发送http请求

 ```python
  def sendHttp(url):
      headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:61.0) Gecko/20100101 Firefox/61.0",
                 "Referer": url}
      r = requests.get(url, headers=headers)  # 增加headers, 模拟浏览器
      return r.text
 ```

* re库    ->正则处理

 ```python
   data = re.findall(r'(\w*[0-9]+)\w*', i)
    dataNum = int(data[0])
 ```

* 文件下载

 ```python
    r = requests.get(url, headers=headers, stream=True)  # 增加headers, 模拟浏览器  stream=分块下载
      if r.status_code == 200:
          with open(r'{}\{}第{}页.jpg'.format(path, title, page), 'ab') as f:
              for data in r.iter_content():
                  f.write(data)
 ```

  <!--more-->

* threading库    ->多线程

 ```python
   thread = threading.Thread(target=get_detail_pic_list,
                                                args=(pic_list[0]["link"], pic_list[0]["title"]))
 ```

  

* Workbook库   ->导出xlsx

 ```python
  '''
  conda install -c conda-forge openpyxl
  '''
  
  from openpyxl import Workbook
  
   wb = Workbook()  # 打开 excel 工作簿
      ws1 = wb.active  # 激活
      ws1.title = lang_name  # 给予文件标题
      ws1.append(row)    #输入列表的列表
      ws2.save(r'{}\{}的{}岗位信息.xlsx'.format(path, i, lang_name)) # 输出文件
 ```

*  os库   ->用来操作系统命令 如文件夹创建

 ```python
      if not os.path.exists(path):  # 创建目录文件 自动递归创建文件夹
          os.makedirs(path)
 ```

  

* scrapy爬虫框架(推荐) 

* selenium + phantomjs 模拟点击按钮

