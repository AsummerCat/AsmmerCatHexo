---
title: vue笔记vuex状态管理模式权限控制
date: 2020-06-25 22:06:15
tags: [vue]
---

# vue笔记vuex状态管理模式权限控制

vuex是一个专门为vue.js开发的状态管理模式
集中式存储管理应用的所有组件状态

## 安装
```
npm install vuex --save
```

<!--more-->
### 修改main.js 导入vuex
```
inport Vuex from 'vuex'

Vue.use(Vuex);
```
### vuex的格式
```
export default new Vuex.Store({
    //data数据
    state:{
        //用户数据啥的
    },
    //计算属性
    getters:{
        //在组件中是通过this.$store.getters.方法名称,来调用getters的
    },
    //方法 在mutations处理state的状态
    mutations:{
        //该区域的方法名来调用this.$store.commit('方法名',state)
    },
    //异步方法
    actions:{
        //在组件中是通过this.$store.dispatch('方法名',user);来调用actions的
        //调用mutations的方法
    },
    //模块
    modules:{
        //这里的话就是如果数据很复杂,state对象很臃肿,可以配置区分模块
        
    }
})

```
<!--more-->
#### modules的用法
```
const moduleA = {
    state:{...},
    getters:{...},
    mutations:{...},
    actions:{...}
}
const moduleB = {
    state:{...},
    getters:{...},
    mutations:{...},
    actions:{...}
}

modules:{
    a:moduleA,
    b:moduleB
}

##使用
store.state.a //=>moduleA的状态
store.state.b //=>moduleB的状态

```

### 创建Vuex配置文件
在`src`目录下创建一个名为`store`的目录并新建一个名为`index.js`的文件用来配置Vuex,代码如下:

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex);

//全局state对象,用来保存所有组件的公有数据
const state={
    //定义一个user对象
    //在组件中是通过this.$store.state.user来获取
    user:{
        username:'',
    }
};

//实时监听state值的最新状态,注意这里的getters,可以理解为计算属性
const getters={
    //在组件中是通过this.$store.getters.getUser来获取的
    getUser(state){
        return state.user;
    }
};

//定义改变state初始值的方法,这里是唯一可以改变state的地方,缺点是只能同步执行
const mutations={
    //在组件中是通过this.$store.commit('uodateUser',user);方法来调用的
    updateUser(state,user){
        state.user=user;
    }
};

//定义出发mutations里函数的方法,可以异步执行mutations里的函数
const actions={
     //在组件中是通过this.$store.dispatch('asyncUpdateUser',user);来调用actions的
     asyncUpdateUser(context,user){
         context.commit('updateUser',user);
     }
};

//这里就是类似于modules 来分模块
export default new Vuex.store({
 state,
 getters,
 mutations,
 actions
})
```
### 在main.js中导入刚刚写的store/index.js
```
import Vuex from 'vuex'
import store form './store'


Vue.use(Vuex);


//在vue对象里引用到这个store
new Vue({
 el: '#app',
 router,
 //引入vuex:
 store,
 reder:h=>h(App)
})

```
这样的话 this.$store 就能指到这个全局vue对象里了


## 使用
在登录成功后的ajax请求返回里加入:

### 登录
```
//往vue中放入一个对象
 let vm=this;
 vm.$store.dispatch('asyncUpdateUser',user);
```
### 进入首页
```
获取用户名
{{this.$store.state.user}}
```

 `注意` 虽然是保存了vuex 但是还是要用路由登录校验做处理
`在路由前配置beforeEach`

## 页面使用 有两种引入方式
### 第一种直接点属性
```
//异步执行
this.$store.dispatch('方法名',user);
//同步执行
this.$store.commit('方法名',user);
//获取属性
this.$store.state.属性名
//计算属性
this.$store.getters.方法名

```
### 使用import
import {mapGetters,mapState} from 'vuex'
这里表示 可以导入store里的哪些内容
```
import {mapGetters,mapState} from 'vuex'

let getter=mapGetters(['方法A','方法B']);
let state =mapState(['属性1','属性2'])
```



## 解决浏览器刷新后Vuex数据消失问题
### 原因:
```
vuex的存储的数据是在页面中的,相当于我们定义的全局变量,
刷新之后,里面的数据就会恢复到初始状态,会把用户信息移除

```
### 解决方案:
```
监听页面是否刷新,如果页面刷新,将store对象存入到sessionStorage中.
页面打开之后,判断sessionStorage中是否存在state对象,
如果存在,则说明页面是被刷新过的,将sessionStore中存的数据取出来给vuex中的state赋值.
如果不存在,说明第一次打开,则去vuex中定义的初始值
```
### 修改代码 添加监听器
在`App.vue`中添加监听代码
window.addEventListener 添加监听方法
```
export default{
    name: 'App',
    mounted(){
        window.addEventListener('unload',this.saveState);
    },
    methods:{
        saveState(){
            sessionStorage.setItem('state',JSON.stringify(this.$store.state));
        }
    }
    
}

```
并且修改./store/index.js中的获取全局state的方法
```
const state=sessionStorage.getItem('state')? JSON.parse(sessionStorage.getItem('state')):{
     user:{
        username:''
    }
}
```

### 关于 sessionStorage相关操作
```
赋值:
sessionStorage.setItem(key,value);
读取值:
sessionStorage.getItem(key);
清除所有key:
sessionStorage.clear();
清除指定key:
sessionStorage.removeItem(key);
获取sessionStorage的项目数:
sessionStorage.length;
```