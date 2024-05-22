---
title: 安卓调试adb加Appium实现爬虫三
date: 2024-05-22 23:37:32
tags: [安卓,爬虫,adb,Appium]
---
# Appium的基础命令定位方式
## 1、ID定位
```
ID定位需要使用页面的resource id属性，在java中的方法对应：findElementById()
```
<!--more-->

//QQ登录页面，登录按钮定位
```
driver.findElementById("com.tencent.mobileqq:id/btn_login").click();
```
## 2、name定位

name定位需要对应页面的text属性，由于findElementByName在appium版本1.5后就被废除了，需要使用xpath来定位text属性结合定位（往后看）。

//QQ登录页面，登录按钮定位
```
driver.driver.findElementByName("登录").click()
```
## 3、class定位

classname定位是根据元素类型来进行定位，但是实际情况中很多元素的classname都是相同的，如QQ登录页面中的用户名和密码的class属性值都是：“android.widget.EditText”，因此只能定位第一个元素也就是用户名，而密码输入框就需要使用其他方式来定位，这样其实很鸡肋，一般情况下如果有id就不必使用classname定位。

在java中对应的方法是：findElementByClassName()

//QQ登录页面，输入用户名
```
driver.findElementByClassName("android.widget.EditText").sendKeys("123456789");
```
## 4、Xpath定位

xpath定位是一种路径定位方式，主要是依赖于元素绝对路径或者相关属性来定位，但是绝对路径xpath执行效率比较低（特别是元素路径比较深的时候），一般使用比较少。通常使用xpath相对路径和属性定位。

可以通过Inspector直接获取页面元素的xpath路径，在java中对应的方法是：findElementByXPath（）

//QQ登录页面，输入用户名
```
driver.findElementByXPath("//android.widget.RelativeLayout/android.widget.RelativeLayout[2]/android.widget.RelativeLayout[1]/android.widget.EditText[1]").click();
```
## 5、Uiautomator定位

UIAutomator元素定位是 Android 系统原生支持的定位方式，虽然与 xpath 类似，但比它更加好用，且支持元素全部属性定位.定位原理是通过android 自带的android uiautomator的类库去查找元素。

Appium元素定位方法其实也是基于Uiautomator来进行封装的。

### id定位
//QQ登录页面，登录按钮定位
```
driver.findElementByAndroidUIAutomator("resource-id(\"com.tencent.qqlite:id/btn_login\").click();
```
### text定位
//QQ登录页面，登录按钮定位
```
driver.findElementByAndroidUIAutomator("text(\"登 录\")").click();
```

### 滑动页面
```
from appium.webdriver.common.touch_action import TouchAction

def test_scroll_down(driver):

    screen = driver.get_window_size()

    action = TouchAction(driver)

    action.press(x=screen['width']/2,y=screen['height']/2)

    action.move_to(x=0,y=-screen['height']/10)

    action.release()

    action.perform()
```

### py的测试案例
```
# 导入webdriver
from appium import webdriver
# 初始化参数
desired_caps = {
    'platformName': 'Android',  # 被测手机是安卓
    'platformVersion': '10',  # 手机安卓版本
    'deviceName': 'xxx',  # 设备名，安卓手机可以随意填写
    'appPackage': 'tv.danmaku.bili',  # 启动APP Package名称
    'appActivity': '.ui.splash.SplashActivity',  # 启动Activity名称
    'unicodeKeyboard': True,  # 使用自带输入法，输入中文时填True
    'resetKeyboard': True,  # 执行完程序恢复原来输入法
    'noReset': True,  # 不要重置App，如果为False的话，执行完脚本后，app的数据会清空，比如你原本登录了，执行完脚本后就退出登录了
    'newCommandTimeout': 6000,
    'automationName': 'UiAutomator2'
}
# 连接Appium Server，初始化自动化环境
driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)
# 设置等待时间，如果不给时间的话可能会找不到元素
driver.implicitly_wait(5)
# 点击搜索框
driver.find_element_by_id("expand_search").click()
# 输入“泰坦尼克号”
driver.find_element_by_id("search_src_text").send_keys("泰坦尼克号")
# 键盘回车
driver.keyevent(66)
# 因为它搜索完后就直接退出app了，看不到搜索结果页，所以我给了一个让他停下的方法
input('**********')
# 退出程序，记得之前没敲这段报了一个错误 Error: socket hang up 啥啥啥的忘记了，有兴趣可以try one try
driver.quit()

```