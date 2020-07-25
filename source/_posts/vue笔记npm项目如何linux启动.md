---
title: vue笔记npm项目如何linux启动
date: 2020-06-25 22:06:37
tags: [vue]
---

# vue笔记npm项目如何linux启动

```
1. npm run build

2.上传到tomcat里的webapps目录里

3. 根据文件夹名称与端口号访问

```

<!--more-->
### 分解
* 1.修改config文件夹下的`index.js`文件   
![index.js](/img/2020-07-25/1.png)
* 修改build `文件夹下的 webpack.prod.conf.js 文件`  
![conf.js](/img/2020-07-25/2.png)  
添加红框部分   
![添加红框部分 ](/img/2020-07-25/3.png)    
* 修改build 文件夹下的` utils.js` 文件  
![utils.js](/img/2020-07-25/4.png)   
* 修改路由
```
修改这个可以解决部署到服务器上后打开页面空白的bug
打包的时候一定要将mode 改为 hash 模式
```
![修改路由](/img/2020-07-25/5.png)   

* `npm run build`
```
会生成一个dist文件夹，里面有个static文件夹和index.html文件
```
![npm run build](/img/2020-07-25/6.png)   

* 移动到tomcat的`webapps`目录里
```
dist文件夹内容移动到webapps里

```