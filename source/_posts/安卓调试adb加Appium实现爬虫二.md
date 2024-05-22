---
title: 安卓调试adb加Appium实现爬虫二
date: 2024-05-22 23:37:00
tags: [安卓,爬虫,adb,Appium]
---
# appium基础使用

## APP的定位方法：【参考web的selenium定位，都差不多】



## 案例一 QQ登录:
新建Python文件
创建unittest单元测试类，并添加setup及teardown函数
对类MyTestCase添加setUp函数（这是测试用例执行前的准备动作，负责告诉appium监听那个端口、与那个手机连接、安装那个应用等信息）
对类MyTestCase添加TearDown函数（这是测试用例执行结束后的动作，可以执行保存执行结果等操作）
添加test开头的方法，编写自动化测试用例

<!--more-->
```
import unittest
import time
from appium import webdriver
from appium.webdriver.common.appiumby import AppiumBy
from appium.options.common import AppiumOptions

class MyTestCase(unittest.TestCase):

    def setUp(self):
        # super().setUp()
        print('selenium version = ', selenium.__version__)
        option = AppiumOptions()
        option.set_capability("platformName","Android")
        option.set_capability("platformVersion","5.1.1")
        option.set_capability("deviceName","Android Emulator")
        option.set_capability("noReset",True)
        option.set_capability("appPackage","com.tencent.qqlite")
        option.set_capability("appActivity","com.tencent.mobileqq.activity.SplashActivity")

        self.driver = webdriver.Remote('http://localhost:4723/wd/hub', options=option)
 
 
    def testQQLogin(self):

        time.sleep(2)
        self.driver.find_element(AppiumBy.ID,"com.tencent.qqlite:id/btn_login").click()

        time.sleep(5)
        self.driver.find_element(AppiumBy.XPATH'//android.widget.EditText[@content-desc="请输入QQ号码或手机或邮箱"]').send_keys("2572652583")
        time.sleep(5)
        self.driver.find_element(AppiumBy.ID,'com.tencent.qqlite:id/password').send_keys("123456789")
        time.sleep(5)
        self.driver.find_element(AppiumBy.ID,'com.tencent.qqlite:id/login').click()

    def tearDown(self):
        self.driver.quit()
 
 
if __name__ == '__main__':
    unittest.main()
```
## ADB获取包名
可以获取到真机聚焦页面的包名
```
adb shell dumpsys window windows | grep mCurrentFocus
```

## Appium-Capability
Capability的功能是配置Appium会话。

他们告诉Appium服务器您想要自动化的平台和应用程序。DesiredCapabilities是一组设置的键值对的集合，其中键对应设置的名称，而值对应设置的值。（如："platformName": "Android"）DesiredCapabilities主要用于通知Appium服务器建立需要的Session。

```adb连接上启动appium服务后 ->New Session Window```
### New Session Window设置
Automatic Server 本地AppiumServer服务
Custom Server：例如，如果要针对运行在网络中另一台计算机上的Appium服务器启动Inspector会话，这很有用。
Sauce Labs：如果您无法访问机器上的iOS模拟器，则可以利用Sauce Labs帐户在云中启动Appium会话。
TestObject：您还可以利用TestObject的真实设备云来进行真机测试。
### 定位相关的可以看下这里
点击上图右下角的Start Session，启动一个新的会话，如果你的android模拟器或者真机已经连接上ADB，则可以直接抓取到设备的页面
```
https://zhuanlan.zhihu.com/p/141439504
```