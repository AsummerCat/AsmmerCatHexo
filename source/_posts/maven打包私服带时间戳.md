---
title: maven打包私服带时间戳
date: 2020-06-12 14:42:20
tags: [maven,打包]
---

# maven打包私服带时间戳

在本地打包的时候   
会拉取私服的版本+本地打包的版本  
导致两个jar相同 可能会导致一些问题  

出现情况:
```
lib文件中出现:
xxx_20200607.jar
xxx.jar
```
## 出现的原因
```
私服上打包的时候 没有发布正式版本,发的都是测试版本
maven会自带时间戳上传

<parent>
        <artifactId>lanxingman-parent</artifactId>
        <groupId>com.lanxingman.project</groupId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <version>0.0.1-RELEASE</version>
    <artifactId>lanxingman-base-project</artifactId>   

SNAPSHOT版本是快照保本–不稳定，尚处于开发中的版本，maven会自动加时间戳

RELEASE 发布版本–稳定版本
```

## 解决方案
在本地打包中

只需要在打war的模块中添加
```java
注意
可通过在pom.xml中包含如下插件的方式解决。

<plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.2.3</version>
                <configuration>
                    <webResources>
                        <resource>
                            <directory>src/main/webapp</directory>
                        </resource>
                    </webResources>
                    <outputFileNameMapping>@{artifactId}@-@{baseVersion}@.@{extension}@</outputFileNameMapping>
                </configuration>
            </plugin>
        </plugins>

 
```

