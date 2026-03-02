---
title: pinia的使用全局状态管理
date: 2026-03-02 11:15:01
tags: [vue]
---

# pinia的使用 全局状态管理
```
    npm install pinia
```

<!--more-->

## 引入pinia

在`app.vue中加入`

```
import {createApp} from `vue`
import App from './App.vue'
import {createPinia} from `pinia`

const app =createApp(App)
//创建pinia
const pinia=createPinia()
app.use(pinia)

app.mount('#app')

```

## 创建store文件夹

在里面对应的文件的ts中写入
```
    比如 count.ts

    import {defineStore} from 'pinia'

    //需要堆外暴露
    export const useXXXstore =defineStore( 
     名称count,{
     //这里可以直接写store的函数 ,并且可以使用this
     actions:{
        add(){
        this.sum+1
        }
     }
     
     //存储数据的地方->回调函数
     state(){
         return {
         //参数值
             sum:6
         }
       }
     }
    )


    --> 业务的count.vue使用
    import { useXXXstore } from `@/store/count.ts`
    //如果希望获取到的是相应对象而不用store.属性,直接用属性的话 需要加载这个
    import {storeToRefs} from 'pinia'
    constcount store=useXXXstore()

    const {sum}=storeToRefs(store)
    console.log(sum)

    console.log(store.sum)

    //可以直接修改数据 依次增加1
    store.sum+=1 


    //也可以使用批量修改 $patch
    store.$patch({
        sum:888,
        age:16
    })

    //调用函数
    store.add()

    //监控store中数据变化
    store.$subscribe((mutate,state)=>{
        localStorage.setItem('sum',state.sum)
    })

```