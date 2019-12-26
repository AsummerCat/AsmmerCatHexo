title: python爬虫需要的工具
date: 2019-12-26 10:35:33
tags: [python,爬虫]

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

  

* 