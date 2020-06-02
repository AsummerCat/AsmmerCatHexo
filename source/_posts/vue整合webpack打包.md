---
title: vue整合webpack打包
date: 2020-06-02 19:50:09
tags: [vue,webpack]
---

# Webpack

## 创建一个vue项目
 使用webpack创建
 ```
 vue init webpack 项目名称
 ```
 ## 初始化并运行
 ```
 cd xxx项目目录
 npm install
 npm run dev
 ```

<!--more-->

 # webpack

 ## 安装webpack
 ```
 npm install webpack -g
 npm install webpack-cli -g
 
 查看版本 webpack-cli -v
 ```
 ## 项目结构
 创建webpack.config.js配置文件
 ```
-  entry: 入口文件,指定webpack用哪个文件作为项目的入口
-  output: 输出,指定webpack吧处理完成的文件放置到指定路径
-  module: 模块,用于处理各种类型的文件
-  plugins: 插件,如:热更新,代码重用等
-  resolve: 设置路径指向
-  watch: 监听,用于设置文件改动后直接打包
 
 
 ```


## 使用webpack的js等文件
1.创建项目  
2.创建一个名为`modules`的目录,用于JS模块的等资源模块  
3.在modules下创建模块文件,如`hello.js`,用于编写JS模块相关代码
```
//暴露多个方法:sayHi
exports.sayHi=function(){
    document.write("<div>hello webpack</div>");
}
exports.sayHi2=function(){
    document.write("<div>hello 1</div>");
}
```
4.在modules下创建一个名为main.js的入口文件,用于打包时设置entry属性
```
//require 导入一个模块,就可以调用这个模块中的方法了
var hello=require("./hello");
hello.sayHi();
```
5.在项目目录下创建webpack.config.js配置文件,使用webpack命令打包
```
module.export={
<!--入口-->
    entry:"./modules/main.js",
    <!--输出的位置--> 
    output:{
        filename: "./js/bundle.js"
    }
}

```

### 静态文件
```
放入static文件夹才可以访问
```