---
title: vue笔记利用node创建web服务器
date: 2020-06-25 22:05:37
tags: [vue]
---

# vue笔记利用node创建web服务器
## 创建node项目 并安装`express`
```
npm -i express
```
## 项目build 生成静态资源
生成`/dist`目录
```
npm run build

```
<!--more-->

## 修改配置文件
修改`app.js`
```
const express=require('express')
const app=express()

app.use(express.static('./dist'))

app.listen('80',()=>{
    console.log('server running at 80端口');
})

```

## 开启Gzip压缩
修改`app.js`
注意 引用一定要写在静态资源前面不然无法使用压缩
```
const compression=require('compression')
//注册Gzip压缩 引用一定要写在静态资源前面不然无法使用压缩
app.use(compression());

app.use(express.static('./dist'))


```

## 设置Https证书配置
修改`app.js`
```
const https=require('https')
const fs = require('fs')
const options={
    cert: fs.readFileSync('./full_chain.pem'),
    key: fs.readFileSync('./private.key')
}

https.createServer(options,app).listen(443);
```

## 安装 pm2 使其在后台运行系统

```
1. 在服务器中安装pm2: npm i pm2 -g
2. 启动项目: pm2 start 脚本(这里就是main.js) --name 自定义名称
3. 查看运行的项目: pm2 ls
4. 重启项目: pm2 restart 自定义名称
5. 停止项目: pm2 stop 自定义名称
6. 删除项目: pm2 delete 自定义名称

```