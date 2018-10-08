---
layout: hexo
title: hexo->next 代码块复制功能
date: 2018-10-05 14:14:10
tags: hexo
---

# 1.下载 clipboard.js
下载地址：

[clipboard.js](https://raw.githubusercontent.com/zenorocha/clipboard.js/master/dist/clipboard.js)  

[clipboard.min.js](https://raw.githubusercontent.com/zenorocha/clipboard.js/master/dist/clipboard.min.js)           推荐  

---

<!--more-->

# 2.使用 
保存文件`clipboard.js / clipboard.min.js` ，目录如下：
`.\themes\next\source\js\src`

#### 2.1 创建初始JS
在`.\themes\next\source\js\src`目录下，创建`clipboard-use.js`，文件内容如下：

```
/*页面载入完成后，创建复制按钮*/
!function (e, t, a) { 
  /* code */
  var initCopyCode = function(){
    var copyHtml = '';
    copyHtml += '<button class="btn-copy" data-clipboard-snippet="">';
    copyHtml += '  <i class="fa fa-globe"></i><span>copy</span>';
    copyHtml += '</button>';
    $(".highlight .code pre").before(copyHtml);
    new ClipboardJS('.btn-copy', {
        target: function(trigger) {
            return trigger.nextElementSibling;
        }
    });
  }
  initCopyCode();
}(window, document);

```

---

#### 2.2添加样式

在`.\themes\next\source\css\_custom\custom.styl`样式文件中添加下面代码：

```
//代码块复制按钮
.highlight{
  //方便copy代码按钮（btn-copy）的定位
  position: relative;
}
.btn-copy {
    display: inline-block;
    cursor: pointer;
    background-color: #eee;
    background-image: linear-gradient(#fcfcfc,#eee);
    border: 1px solid #d5d5d5;
    border-radius: 3px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    -webkit-appearance: none;
    font-size: 13px;
    font-weight: 700;
    line-height: 20px;
    color: #333;
    -webkit-transition: opacity .3s ease-in-out;
    -o-transition: opacity .3s ease-in-out;
    transition: opacity .3s ease-in-out;
    padding: 2px 6px;
    position: absolute;
    right: 5px;
    top: 5px;
    opacity: 0;
}
.btn-copy span {
    margin-left: 5px;
}
.highlight:hover .btn-copy{
  opacity: 1;
}
```
---

#### 2.3 引用

在`.\themes\next\layout\_layout.swig`文件中，添加引用（注：在 `swig` 末尾或 `body` 结束标签`（</body>）`之前添加）：

```
<!-- 代码块复制功能 -->
  <script type="text/javascript" src="/js/src/clipboard.min.js"></script>  
  <script type="text/javascript" src="/js/src/clipboard-use.js"></script>
```
---