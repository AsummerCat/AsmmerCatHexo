---
title: 安卓调试adb加Appium实现爬虫一
date: 2024-05-22 23:35:54
tags: [安卓,爬虫,adb,Appium]
---
## 1.首先下载安卓模拟器
`mumu.163.com`

## 2.安装java的jdk
新增环境变量
## 3.安装android SDK
`https://dl.google.com/android/installer_r24.4.1-windows.exe`
我们只需要下SDK Tools，而不需要整个Android Studio都下载下来
注意安卓的版本 只要勾选
```
Android SDK Tools
Android SDK Platform-tools
Android SDK Build-tools
```
并且选择对应的安卓版本 例如 Android 6.0(API 23)
<!--more-->

### 3.1 新增环境变量
```
ANDROID_HOME: %ANDROID_HOME%\platform-tools %ANDROID_HOME%\tools
```
### 3.2 测试安卓环境
```
在cmd界面中输入 adb
```

## 4.Appium下载
```
链接：github.com/appium/appium-desktop/releases
```

## 5.安装py环境
```
建议直接使用python3.8
```

### 5.1 安装appium的python依赖
```
这里丢一个官方github：github.com/appium/python-client

值得按照官方的教程走即可，我这里pip

pip install -i https://pypi.douban.com/simple  Appium-Python-Client
```

## 6.测试appium和adb的环境
首先adb需要先连接到mumu，一般来说，直接在cmd界面输入：

`adb connect localhost:7555`
该命令的意思是，adb链接到默认端口为7555端口的mumu模拟器

```
C:\Users>adb connect localhost:7555
connected to localhost:7555
```
然后输入adb devices
```
C:\Users>adb devices
List of devices attached
localhost:7555  device
```
如果长这样，就意味着 安卓tools的adb+模拟器没问题了。

### 6.1 打开Appium的desktop界面
直接点击“startServer”即可 页面显示:`0.0.0.0 : 4723`

### 6.2 打开py链接调试
打开python的编辑器，输入如下代码进行测试
```
# coding=utf-8

from appium import webdriver

desired_caps = {
  'platformName': 'Android',
  'deviceName': 'xxxx',
  'platformVersion': '6',
  'appPackage': 'com.android.settings',
  'appActivity': 'com.android.settings.Settings'
 }

driver = webdriver.Remote('http://127.0.0.1:4723/wd/hub', desired_caps)
```
该代码的意思是，自动在模拟器中打开设置，如果上面配置均没有问题，那么就意味着配置配置到这里，一切环境都搞定啦



