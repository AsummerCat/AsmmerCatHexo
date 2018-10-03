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



# 1.安装Node.js

```
官网: https://nodejs.org/en/

在官网下载v6.9.4 LTS版本，安装即可.我们用它来生成静态网页.
```

---

# 2.安装hexo
输入以下命令安装:  

```
sudo npm install -g hexo

```
---

# 3.安装成功初始化目录结构

```
新建一个目录/Users/yanzi/work/hexo

然后进到hexo目录下执行hexo init,完成后初始化目录

接着

然后在执行npm install, 多了个db.json文件 
```

---

# 4.基本命令

```
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

# 5.关联github


在github上新建个仓库，名为yourname.github.io,yourname是github的用户名，这个规则不能变.然后新建一对ssh的key,将公钥添加到github
>请参考: https://help.github.com/articles/connecting-to-github-with-ssh/  

添加SSH keys之后，就可以使用git为后缀的仓库地址，本地push的时候无需输入用户名和密码.

>注意:考虑到大家不止一个github，此处如果不这样处理，使用https的仓库地址，再接下来部署时往往会出现不让输入github用户名和密码的问题!

编辑本地hexo目录下的_config.yml文件,搜索deploy关键字，然后添加如下三行:

```
deploy:
  type: git
  repository: http://github.com/AsummerCat/AsummerCat.github.io.git
  branch: master

```
**注意:每个关键字的后面都有个空格**  

---

# 6.使用next主题  
<font color="red">**站点配置文件 :  根目录下的_config.yml 配置文件**</font>  
<font color="red">**主题配置文件 :  next主题下的_config.yml 配置文件**</font>

>tips:可以参考[next文档](http://theme-next.iissnan.com/getting-started.html#clone)  安装

默认的主题不太好看，我们使用点赞最高的next主题。 
在博客根目录下执行:  
`git clone https://github.com/iissnan/hexo-theme-next themes/next`

#### 6.1启动主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 ， 找到 theme 字段，并将其值更改为 next。

`theme: next`

#### 6.2设置语言

编辑 站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

`language: zh-Hans`

#### 6.3 设置菜单

设定菜单内容，对应的字段是 menu。 菜单内容的设置格式是：item name: link。其中 item name 是一个名称，这个名称并不直接显示在页面上，她将用于匹配图标以及翻译。

```
NexT 默认的菜单项有（标注  的项表示需要手动创建这个页面）：

键值	设定值	显示文本（简体中文）
home	home: /	主页
archives	archives: /archives	归档页
categories	categories: /categories	分类页 
tags	tags: /tags	标签页 
about	about: /about	关于页面 
commonweal	commonweal: /404.html	公益 404 
```

例如:

```
menu:
  home: / || home
  about: about || user
  tags: tags || tags
  categories: categories || th
  archives: archives || archive

menu_icons:
  enable: true
```

#### 6.4设置作者昵称
编辑 站点配置文件， 设置 author 为你的昵称。

`author: 夏天的猫`

#### 6.5设站点描述
编辑 站点配置文件， 设置 description 字段为你的站点描述。站点描述可以是你喜欢的一句签名:)

#### 6.6集成常用的第三方服务

>[集成第三方服务的文档](http://theme-next.iissnan.com/third-party-services.html#wei-sousuo)

##### 6.6.1 搜索功能Local Search

```
添加百度/谷歌/本地 自定义站点内容搜索

安装 hexo-generator-searchdb，在站点的根目录下执行以下命令：

$ npm install hexo-generator-searchdb --save
编辑 站点配置文件，新增以下内容到任意位置：

search:
  path: search.xml
  field: post
  format: html
  limit: 10000
编辑 主题配置文件，启用本地搜索功能：

# Local search
local_search:
  enable: true
```
---

## 7 编写文章的说明
#### 7.1 创建一个新文章 生成在 根目录的 /soure/_post/ 中
   `命令:  hexo new 文件名称`
   
#### 7.2标签和分类的使用
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

#### 7.3在文章中的标签和分类使用

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

#### 7.4 文章在首页显示部分内容

在文章中加入:

`<!--more--> 就可以缩略下面部分的内容了`


---

## 8.hexo多电脑同步
很多时候人要在两台电脑上操作hexo随时发表文章，我们将整个hexo目录作为一个私有git仓库，push到http://git.oschina.net.这里没有采用不同branch的管理方式，而是将其作为一个完全独立的私有仓库，托管到码云上.

在hexo目录下执行git init,然后 
注意:因为theme目录下的next也是一个git仓库，而根目录下的git仓库是管不了子仓库的。后边需要对这个next theme作大量定制，因此用submodule的方式管理不合适。我们直接把next下的.git文件夹删除即可，这样就可以被外边的git仓库管理了 
然后进行一次git add .,然后git commit -m "add:first commit".之后push到你在oschina上建的仓库.

这样在另外一台电脑第一次操作时，先把hexo整个仓库git clone下来，然后在hexo目录下执行  
npm install hexo --save 
再执行一次   npm install  然后


```
hexo clean
hexo g
hexo s
```

在本地测试下。每次在不同电脑上发表博文时，先git pull一次，然后再写文章。


