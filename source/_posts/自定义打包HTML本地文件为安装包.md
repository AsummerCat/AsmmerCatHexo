---
title: 自定义打包HTML本地文件为安装包
date: 2024-05-25 17:54:03
tags: [静态文件打包,Pake]
---

# 中文操作文档
https://github.com/tw93/Pake/blob/master/bin/README_CN.md

https://tauri.app/v1/guides/getting-started/prerequisites/
## 安装Microsoft Visual Studio C++ Build Tools
```
 Build Tools for Visual Studio 2022
.
安装模块
1.MSVC V143 -VS 2022 C++
2. Windows 10 SDK
```
<!--more-->
## 安装rust
https://www.rust-lang.org/tools/install
```
输入选择默认安装 :1

安装后 cmd 输入:
rustup default stable-msvc

有返回表示安装完成
```

## 安装依赖
```
npm install pake-cli -g
```


## pake打包静态文件
pake-cli >= 2.3.3，不确定自己是什么版本可以用pe --version确定

### 执行
执行下面的命令打包静态文件，启动文件一般是index.html，当然也可能是其它的，看前端咯。
```
pake ./AriaNg-1.3.6/index.html  --name html-test --use-local-file

--use-local-file
当 url 为本地文件路径时，如果启用此选项，则会递归地将 url 路径文件所在的文件夹及其所有子文件复
制到 Pake 的静态文件夹。默认不启用。
```

### PS:注意如果打包 依赖有下载问题
手动处理
```
C:\Users\用户\AppData\Local\tauri

新建这个目录
C:\Users\BOTONG\AppData\Local\tauri\WixTools

将https://objects.githubusercontent.com/github-production-release-asset-2e65be/17723789/6aaeda80-da25-11e9-8564-82dd8c5115cd?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240523%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240523T075156Z&X-Amz-Expires=300&X-Amz-Signature=12706feadc88340dcd75aa755c9a0df1f2f3c5583013f418fc1a1f276e3d19b9&X-Amz-SignedHeaders=host&actor_id=25723461&key_id=0&repo_id=17723789&response-content-disposition=attachment%3B%20filename%3Dwix311-binaries.zip&response-content-type=application%2Foctet-stream

下载后的内容解压进去就可以免下载了
```

## 测试案例
```
pake ./web/index.html --use-local-file  --name jc  --fullscreen --always-on-top --disabled-web-shortcuts --hide-title-bar --width 1920 --height 1078
```