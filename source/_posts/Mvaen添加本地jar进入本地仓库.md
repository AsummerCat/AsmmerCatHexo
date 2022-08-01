---
title: Mvaen添加本地jar进入本地仓库
date: 2020-06-02 19:43:38
tags: [java,maven]
---

# Mvaen添加本地jar进入本地仓库


```

mvn install:install-file -DgroupId=com.dm -DartifactId=dm7 -Dversion=7.0 -Dpackaging=jar -Dfile=C:\Users\Administrator\Desktop\drive\drive\Dm7JdbcDriver-1.0.jar


mvn install:install-file -Dfile="jar包的位置" -DgroupId=jar包的groupId坐标 -DartifactId=jar包的artifactId坐标 -Dversion=jar包的version坐标 -Dpackaging=jar

```