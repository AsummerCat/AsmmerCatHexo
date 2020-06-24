---
title: vue笔记ele后台前端解决方案
date: 2020-06-24 22:31:50
tags: [vue]
---

#  vue笔记ele后台前端解决方案 
vue-element-admin

可以快速搭建后台的前端框架
[vue-element-admin官网文档地址](https://panjiachen.github.io/vue-element-admin-site/zh/guide/)

[vue-element-admin官网git地址](https://github.com/PanJiaChen/vue-element-admin)

## 分页插件
Pagination插件   
基于ele实现   
[测试ele地址](https://panjiachen.github.io/vue-element-admin)  
[参考地址](https://github.com/PanJiaChen/vue-element-admin)



<!--more-->
### 完整demo
```
<template>  <div class="block">   <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[100, 200, 300, 400]"//这事下拉框可以选择的，选择一夜显示几条数据
      :page-size="100"  //这是当前煤业显示的条数
      layout="total, sizes, prev, pager, next, jumper"
      :total="400" //这个是总共有多少条数据，把后台获取到的数据总数复制给total就可以了>
    </el-pagination>
  </div>
</template>
<script>
  export default {
    methods: {
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
      }
    },
    data() {
      return {        total:'0',
        currentPage: 4
      };
    }
  }
</script>
```