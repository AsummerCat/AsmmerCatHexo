---
title: hexo多平台发布github及其coding
date: 2018-10-05 09:28:56
tags: hexo
---

# 部署步骤
将博客托管到Coding和GitHub
首先，在本地博客根目录/source/下建立一个名为 CNAME的文件，里面写填入你购买的域名。例如linjingc.com.top, 不需要任何的其他字符，例如“www”,”https”之类。

在本地博客文件夹下的srouce文件夹下新建Staticfile文件，接下来就可以同时向coding和GitHub上传博客了。

向coding提交代码遇到了如下问题怎么办？

remote: Coding 提示: Authentication failed! 认证失败，请确认您输入了正确的账号密码

我是因为配置出现了问题，改成如下样式，即可同时上传github和coding

---

# 关联github


在github上新建个仓库，名为yourname.github.io,yourname是github的用户名，这个规则不能变.然后新建一对ssh的key,将公钥添加到github
>请参考: https://help.github.com/articles/connecting-to-github-with-ssh/  

<!--more-->

添加SSH keys之后，就可以使用git为后缀的仓库地址，本地push的时候无需输入用户名和密码.

>注意:考虑到大家不止一个github，此处如果不这样处理，使用https的仓库地址，再接下来部署时往往会出现不让输入github用户名和密码的问题!

编辑本地hexo目录下的_config.yml文件,搜索deploy关键字，然后添加如下三行:

```
deploy:
  type: git
  repo:
      github: http://github.com/AsummerCat/AsummerCat.github.io.git,master
      coding: http://git.coding.net/AsummerCat/AummerCat.git,master  


```
**注意:每个关键字的后面都有个空格**  

---

# 关联coding
## coding上建仓库
```
 1. 首先，注册个https://coding.net，用户名假设为username,然后新建个仓库(可以是私有哦)，  
名为username,即建立个跟用户名同名的仓库.然后跟github一样，把本地的ssh公钥添加进去。  
然后点击项目，在代码--Pages 服务，在部署来源里选中master 分支

需要注意的是 coding上需要设置自定义域名后 才能继续解析  在pages服务里面
 剩下就是域名解析之类的处理了
```
**需要提醒的是coding上需要银牌会员才可以绑定自定义域名,所以只要绑定下账号就可以免费升级银牌了**

---

## 去除coding广告

去除Coding的广告……  
配置好之后很开心，然而发现，Coding会自动给你的博客加一个滞留好几秒钟的跳转页面，感觉非常非常的不好。
还好除了升级为它的黄金会员之外还有其他解决方法。

只要在博客主页为它打一点点广告就好了（要两个工作日才会通过！）……毕竟托管在上面还是很方便的。

`<div>Hosted by <a href="https://pages.coding.me" style="font-weight: bold">Coding Pages</a>
</div>`
对于Next主题来说，在  
`themes/next/layout/_partials/footer.swig`的文件末尾加入上述代码就可以把Coding要求的小广告加入到主页的页脚位置。

---

# 多平台发布

国内coding 国外github 访问速度快

>在博客根目录下的source路径下新建文件:touch Staticfile #名字必须是Staticfile   
然后hexo g，再hexo d就可以把博客放到coding和github了!

---