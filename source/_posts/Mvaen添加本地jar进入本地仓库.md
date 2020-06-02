---
title: Mvaen添加本地jar进入本地仓库
date: 2020-06-02 19:43:38
tags: [java,maven]
---

# Mvaen添加本地jar进入本地仓库


```

mvn install:install-file -DgroupId=com.dm -DartifactId=dm7 -Dversion=7.0 -Dpackaging=jar -Dfile=C:\Users\Administrator\Desktop\drive\drive\Dm7JdbcDriver-1.0.jar

```