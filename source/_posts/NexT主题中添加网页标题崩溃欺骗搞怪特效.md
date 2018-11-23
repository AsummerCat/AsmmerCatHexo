---
layout: hexo
title: NexT主题中添加网页标题崩溃欺骗搞怪特效
date: 2018-11-23 17:26:37
tags: hexo
---

给网页title添加一些搞怪特效

# crash_cheat.js

<!--more-->

在`next\source\js\src`文件夹下创建`crash_cheat.js`，添加代码：

```
<!--崩溃欺骗-->
 var OriginTitle = document.title;
 var titleTime;
 document.addEventListener('visibilitychange', function () {
     if (document.hidden) {
         $('[rel="icon"]').attr('href', "/img/TEP.ico");
         document.title = '╭(°A°`)╮ 页面崩溃啦 ~';
         clearTimeout(titleTime);
     }
     else {
         $('[rel="icon"]').attr('href', "/favicon.ico");
         document.title = '(ฅ>ω<*ฅ) 噫又好了~' + OriginTitle;
         titleTime = setTimeout(function () {
             document.title = OriginTitle;
         }, 2000);
     }
 });
```

# 引用

在`next\layout\_layout.swig`文件中，添加引用（注：在swig末尾添加）：

```
<!--崩溃欺骗-->
<script type="text/javascript" src="/js/src/crash_cheat.js"></script>

```