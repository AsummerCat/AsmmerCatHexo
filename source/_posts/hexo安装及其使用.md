---
title: hexo安装及其使用
date: 2018-10-03 13:17:33
tags: hexo
---
# 利用hexo用来安装github pages做成静态博客

>参考  
1.[next文档](http://theme-next.iissnan.com/getting-started.html#clone)  
2.[hexo官网文档](https://hexo.io/docs/)
3.[hexo个性化设置(推荐查看)](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)


 
# 安装Node.js

```
官网: https://nodejs.org/en/

在官网下载v6.9.4 LTS版本，安装即可.我们用它来生成静态网页.
```
<!--more-->

---

# 安装hexo
输入以下命令安装:  

```
sudo npm install -g hexo

```
---

# 安装成功初始化目录结构

```
新建一个目录/Users/yanzi/work/hexo

然后进到hexo目录下执行hexo init,完成后初始化目录

接着

然后在执行npm install, 多了个db.json文件 
```

---

# 基本命令

```
npm install hexo -g #安装Hexo
npm update hexo -g #升级
hexo init #初始化博客
  1. 初始化目录   hexo init
  2. 创建一个新文件夹 hexo new page XXX
  3. 创建一个新文件   hexo new  XXX   (生成一个markdown文档)
  4. 清除目录缓存   hexo clean
  5. 生成静态文件   hexo g
  6. 部署			 hexo d
  7. 本地运行测试环境    hexo s
  8. 打包部署可以这么写:  hexo g -d
```

---


## 编写文章的说明
### 创建一个新文章 生成在 根目录的 /soure/_post/ 中
   `命令:  hexo new 文件名称`
   
   ---
   
### 标签和分类的使用
需要显示标签和分类 首先需要设置标签和分类页  

```
1.创建tag目录  
hexo new tags 会默认生成一个index.md

2.修改文件的index.md 改问标签页
---
type: tags
---
这样表明这个是标签页

3.创建categories目录
hexo new categories 会默认生成一个index.md

4.修改文件的index.md 改问标签页
---
type: categories
comments: false 
---
这样表明这个是分类页


需要注意的是 如果需要显示标签的话 需要在 根目录的站点配置文件里 编辑:

#默认的分类
category_map:
  程序: java
  生活: 生活
  
 #默认的标签 
  tag_map:
  JAVA: java
  
```
---

### 在文章中的标签和分类使用

在文章开头加入:

```
---
title: hexo安装及其使用
date: 2018-10-03 13:17:33
tags: hexo    #标签
category: hexo  #分类
---

需要注意的是这么命名都是要加个空格

如果是多个标签和分类的话 可以这么写

tags: [hexo,hexo2,...,...,...]

```

---

### 文章在首页显示部分内容

在文章中加入:

`<!--more--> 就可以缩略下面部分的内容了`


---

