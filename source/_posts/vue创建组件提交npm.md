---
title: vue创建组件提交npm
date: 2020-06-19 16:27:51
tags: [vue,webpack]
---

# vue创建组件 提交到npm

引用的话就直接npm install xxx模块

[demo地址](https://github.com/AsummerCat/zoe-ca-vue)

## 第一步创建一个vue组件项目

```java
vue init webpack-simple xxx-zj
```

初始完项目，按照提示输入项目信息即可，然后 npm install , npm run dev 让项目跑起来，如下图：

## 修改文件目录

#### 1.在src目录下新建components文件夹，然后在此文件夹下建立aSignature.vue文件

```
caSignature.vue 名字也可以是其他，我们写的这个组件就是放在该文件里面，之前的App.vue是用于本地开发，测试用的入口文件，也就是用于 npm run dev  的入口文件。
```

<!--more-->

#### 2.在webpack.config.js同级目录（也是该组件的根目录）下新建 index.js文件， index.js是把Main.vue文件暴露出去的出口。

![](/img/2020-06-19/1.png)

#### 3.修改文件内容，配置

caSignature.vue (注意name的值) 是给其他模块引入的名称

```
export default {
  name: 'zoe-ca-vue',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  }
```

App.vue修改以下内容

```
这里可以自己写测试demo
```

index.js 修改以下内容

```java
import caSignature from './src/components/caSignature'
import _Vue from 'vue'

caSignature.install = Vue => {
  if (!Vue) {
    window.Vue = Vue = _Vue
  }
  caSignature.component(caSignature.name, caSignature)
}
export default caSignature;

```

## 修改package.json

package.json需要修改private字段(private是true的时候不能发布到npm,需设置成false)； 并增加main字段， main字段是require方法可以通过这个配置找

![](/img/2020-06-19/3.png)

```java
  "private": false,
  "main": "./dist/zoe-ca-vue.js",
```

![](/img/2020-06-19/2.png)

## 修改 webpack.config.js

```java
var path = require('path')
var webpack = require('webpack')

// 执行环境
const NODE_ENV = process.env.NODE_ENV;
console.log("-----NODE_ENV===",NODE_ENV);

module.exports = {
  // 根据不同的执行环境配置不同的入口
  entry: NODE_ENV == 'development' ? './src/main.js' : './index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'zoe-ca-vue.js',
    library: 'zoe-ca-vue', // 指定的就是你使用require时的模块名
    libraryTarget: 'umd', // 指定输出格式
    umdNamedDefine: true // 会对 UMD 的构建过程中的 AMD 模块进行命名。否则就使用匿名的 define
  },
```

这里表示dict文件夹里生成的名称和模块名

说明：入口会根据开发环境 ，生产环境切换， main.js 是npm run dev 的入口，就是App.vue， 用于调试/测试我们开发的组件;  index.js是Main.vue， 就是我们开发的组件，我们打包到生产环境打包就只是单纯的 yyl-npm-practice 组件

![](/img/2020-06-19/4.png)

## 修改index.html的js引用路径，因为我们修改了output 的 filename，所以引用文件的名字也得变。

![](/img/2020-06-19/5.png)

到此组件就开发完了，打包下看看， npm run build dist下生成了俩文件，如下：

![](/img/2020-06-19/6.png)

## 发布到npm

#### 1.去 npm 官网注册个账号 https://www.npmjs.com/

#### 2.执行npm logiin

在该组件根目录下的终端（就是 平常输入 npm run dev的地方），运行npm login，会提示输入个人信息，Email是发布成功后，自动给这个邮箱发送一个邮件，通知发布成功，输入完，登录成功。

#### 3.npm publish

发布成功(每次发布的时候packa.json 里面的 version不能一样，不然不能发布出去，手动改下版本就行)

去自己的npm上点击Packages ，就能看到发布的包

大家最好在readme 里面写上组件的使用方法， 说明等等，方便你我他。

## 测试组件

```java
在发正式包之前可以在本地先打一个包，然后测试下有没有问题，如果没问题再发布到npm上。
首先，打包到本地
npm run build
npm pack
npm pack 之后，就会在当前目录下生成 一个tgz 的文件。
打开一个新的vue项目，在当前路径下执行(‘路径’ 表示文件所在的位置)
npm install 路径/组件名称.tgz
然后，在新项目的入口文件（main.js）中引入
```

## 其他模块引用该组件

```
import caSignature from 'zoe-ca-vue'
Vue.use(caSignature)
然后在其他.vue文件里面，直接使用组件 <caSignature/> 即可
```

## npm常用操作

```java
npm config set registry https://registry.npm.taobao.org //设置淘宝镜像

npm config set registry http://registry.npmjs.org //设置官方镜像

npm config get registry //获取镜像地址

npm --force unpublish safe_manage_view //删除npm包

npm login //npm 登录

npm publish //npm 发布包

npm install --save hzm-npm-test //安装包

npm pack //打包
```



