---
title: sublimeText3插件安装及其使用
date: 2018-10-05 01:22:26
tags: [mac,sublimeText]
---

# Bracket Highlighter


功能：代码匹配

简介：可匹配[], (), {}, “”, ”, <tag></tag>，高亮标记，便于查看起始和结束标记

使用：点击对应代码即可

---

<!--more-->

# jQuery

功能：jQ函数提示

简介：快捷输入jQ函数，是偷懒的好方法

---

# ConvertToUTF8

功能：文件转码成utf-8

简介：通过本插件，您可以编辑并保存目前编码不被 Sublime Text 支持的文件，特别是中日韩用户使用的 GB2312，GBK，BIG5，EUC-KR，EUC-JP ，ANSI等。ConvertToUTF8 同时支持 Sublime Text 2 和 3。

使用：安装插件后自动转换为utf-8格式

---


# Sublime Text 主题 – ayu

## 字体
ayu 使用 Roboto Mono 作为主要的字体，强烈建议安装这个字体，这样在文件树就可以显示为等宽字体了。但是，如果你安装这个字体，那么UI主题将降级到 Sublime Text 标准用户界面字体。

---

## 激活
Sublime Text 3
添加这几行代码到你的设置中 `Preferences > Setting – User:`

黑色主体:

js 代码:

```
"theme": "ayu.sublime-theme",
"color_scheme": "Packages/ayu/ayu.tmTheme",
```

亮色主题:

js 代码:

```
"theme": "ayu-light.sublime-theme",
"color_scheme": "Packages/ayu/ayu-light.tmTheme",
```

---

# Sublime Text3 更改默认字体

菜单 -> `Preference`
`Settings - User`
打开 `Preferences.sublime-settings `文件
添加如下内容

```
// 这里可以将 Courier New 修改为自己喜欢的字体
"font_face": "Courier New",
"font_size": 12,
```


---
# 关闭自动更新
添加这几行代码到你的设置中 `Preferences > Setting – User:`

`"update_check":false`

