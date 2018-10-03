---
title:  java服务器根地址
date: 2018-09-20 22:02:14
tag:  java基础
---

```java
request.getSession().getServletContext().getRealPath("/")+File.separator
服务器根地址
${pageContext.request.contextPath }
```



