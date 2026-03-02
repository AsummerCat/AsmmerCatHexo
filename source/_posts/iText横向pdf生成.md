---
title: itext横向pdf生成
date: 2019-09-26 14:42:41
tags: [工具类,java]
---

# itext 横向pdf生成

Document doc = new Document(PageSize.A4);

这个是用PageSize.A4设置的是纵向A4大小

进入 com.itextpdf.text.PageSize的源码 可以看到这些常量



调用PageSize.A4其实就是调用new RectangleReadOnly(595F, 842F);
那么在定义Document时可以这样
Document doc = new Document(new RectangleReadOnly(842F, 595F));这样就设置成A4的横向大小了

