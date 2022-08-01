---
title: HomeBrew两大利器
date: 2019-01-14 15:12:52
tags: [HomeBrew]
---



# HomeBrew基本命令

```
# 安装HomeBrew
$ /usr/bin/ruby -e "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/master/install](https://raw.githubusercontent.com/Homebrew/install/master/install))"

# 搜索软件确切的包名
$ brew search nginx

# 安装软件
$ brew install nginx

# 删除软件
$ brew uninstall nginx

# 更新软件
$ brew update nginx

# 检查已安装，但命令无法执行的软件
$ brew doctor

# 建立环境链接，避免手动配环境变量。
$ brew link nginx
```



<!--more-->

# Cakebrew 管理软件



https://www.cakebrew.com/

# LaunchRocket 启动项管理

omebrew安装的Mysql、Redis、MongoDB，是让它自启动呢，还是手动启动，传统方式需要使用命令行的命令，而使用LaunchRocket则可以在图形界面中进行管理了！

安装方法：

brew tap jimbojsb/launchrocket
brew cask install launchrocket

需要注意的是 如果是命令行启动服务的话  LaunchRocket就无法管理 启动关闭了 所以要先关闭命令行启动方式