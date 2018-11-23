---
title: hexo使用next主题
date: 2018-10-05 09:35:51
tags: hexo
---


# 使用next主题  
<font color="red">**站点配置文件 :  根目录下的_config.yml 配置文件**</font>  
<font color="red">**主题配置文件 :  next主题下的_config.yml 配置文件**</font>

>tips:可以参考  
>1.[next文档](http://theme-next.iissnan.com/getting-started.html#clone)   
>2.[hexo的next主题个性化配置教程](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)  
>3.[打造个性超赞博客Hexo+NexT+GithubPages的超深度优化](https://reuixiy.github.io/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html)

默认的主题不太好看，我们使用点赞最高的next主题。 
在博客根目录下执行:  
`git clone https://github.com/iissnan/hexo-theme-next themes/next`

---

<!--more-->

# 启动主题

与所有 Hexo 主题启用的模式一样。 当 克隆/下载 完成后，打开 ， 找到 theme 字段，并将其值更改为 next。

`theme: next`

---

# 设置语言

编辑 站点配置文件， 将 language 设置成你所需要的语言。建议明确设置你所需要的语言，例如选用简体中文，配置如下：

`language: zh-Hans`

---

# 设置菜单

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

---

# 设置作者昵称
编辑 站点配置文件， 设置 author 为你的昵称。

`author: 夏天的猫`

---

# 设站点描述
编辑 站点配置文件， 设置 description 字段为你的站点描述。站点描述可以是你喜欢的一句签名:)

---

# 集成常用的第三方服务

>[集成第三方服务的文档](http://theme-next.iissnan.com/third-party-services.html#wei-sousuo)

## 搜索功能Local Search

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

# 选择动画背景

在 主题配置文件 中搜索并配置如下信息，最好四选一。

```
# Canvas-nest
canvas_nest: true   # 背景有降落伞

# three_waves
three_waves: false    # 背景有像海浪一样的小球球

# canvas_lines
canvas_lines: false    # 背景有立体蜘蛛网

 canvas_sphere
canvas_sphere: false    # 屏幕中央有一个爆炸状的球球
```
---

# 代码块语法高亮设置
在 站点配置文件 设置

```
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace: true
```

# SEO配置
想要让我们的站点被搜索引擎收录，要提交给他们站点文件。
首先安装两个插件,并生成两个站点文件，sitemap.xml与baidusitemap.xml文件

npm install hexo-generator-sitemap --save -dev
hexo d -g
npm install hexo-generator-baidu-sitemap --save -dev
hexo d -g
在 站点配置文件 配置如下信息:

## SEO优化

sitemap:  
  path: sitemap.xml  
baidusitemap:  
  path: baidusitemap.xml  
新建robots.txt文件，添加以下文件内容，把robots.txt放在hexo站点的source文件下  

```
User-agent: * Allow: /
Allow: /archives/
Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/
Sitemap: http://imbowei.com/sitemap.xml
Sitemap: http://imbowei.com/baidusitemap.xml
```
在 主题配置文件 中配置如下。

```
# Enable baidu push so that the blog will push the url to baidu automatically which is very helpful for SEO
baidu_push: true
```

当然还要去百度站长和谷歌站长验证,bing验证

---

# 自动打开脚本

为了每次新建博文我们可以直接编辑，而不是在一堆文件中找到它再打开。我们需要在博客根目录新建`script`文件夹（已有就不用新建）
在新建的文件夹新建一个.js文件，其中填写的代码如下所示。  

win用户:

```
var spawn = require('child_process').exec;

// Hexo 2.x 用户复制这段
hexo.on('new', function(path){
  spawn('start  "markdown编辑器绝对路径.exe" ' + path);
});

// Hexo 3 用户复制这段
hexo.on('new', function(data){
  spawn('start  "markdown编辑器绝对路径.exe" ' + data.path);
});
```

Mac用户:

```
var exec = require('child_process').exec;

// Hexo 2.x 用户复制这段
hexo.on('new', function(path){
    exec('open -a "markdown编辑器绝对路径.app" ' + path);
});
// Hexo 3 用户复制这段
hexo.on('new', function(data){
    exec('open -a "markdown编辑器绝对路径.app" ' + data.path);
});
```


---