---
title: Html页面特殊符号转义还原
date: 2018-09-20 22:02:14
tags: java
---

### **StringEscapeUtils.unescapeHtml4(zzRewards.getRwTimeCon())**
<!--more-->
```
/**
24     *  apache的StringEscapeUtils进行转义
25      */
26    //&lt;a href='http://www.qq.com'&gt;QQ&lt;/a&gt;&lt;script&gt;
27    System.out.println(org.apache.commons.lang.StringEscapeUtils.escapeHtml(str));
28   
29    /**
30     *  apache的StringEscapeUtils进行还原
31      */
32    //<a href='http://www.qq.com'>QQ</a><script>
33    System.out.println(org.apache.commons.lang.StringEscapeUtils.unescapeHtml("<a href='http://www.qq.com'>QQ</a><script>"));
34 }
```

