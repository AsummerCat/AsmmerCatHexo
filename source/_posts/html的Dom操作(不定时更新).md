---
title: html的Dom操作(不定时更新)
date: 2018-09-20 22:02:14
tags: 前端
---
## 1.弹出确认tip函数

#### *confirm*() 方法用于显示一个带有指定消息和 OK 及取消按钮的对话框。 语法 *confirm*(message) 参数描述 message 要在window 上弹出的对话框中显示的纯文本

```html
<a onclick="return confirmx('确认要删除该专员用户吗？', this.href)">删除</a>
```
<!--more-->
-----

## 2.刷新页面 

reload() 方法用于重新加载当前文档。

```html
  window.location.reload();
```

---

## 3.跳转

window.location.herf="url地址";

```html
self.location.href="/url" 当前页面打开URL页面
location.href="/url" 当前页面打开URL页面
windows.location.href="/url" 当前页面打开URL页面，前面三个用法相同。
this.location.href="/url" 当前页面打开URL页面
parent.location.href="/url" 在父页面打开新页面
top.location.href="/url" 在顶层页面打开新页面
```

-----

## 4.退回上一级

go() 方法可加载历史列表中的某个具体的页面。

（-1上一个页面，1前进一个页面)

```
window.history.go(-1);
```

