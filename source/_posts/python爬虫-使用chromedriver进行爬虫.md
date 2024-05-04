---
title: python爬虫-使用chromedriver进行爬虫
date: 2024-05-04 16:01:48
tags: [python, 爬虫,chromedriver]
---
进行唤起浏览器模拟用户行为进行抓包
需要注意的是: chromedriver 要放在py的路径里
<!--more-->
```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

## 需要下载对应chromedriver.exe进行打开
if __name__ == '__main__':
    chrome_driver_path = "C:\\Users\\Administrator\\Desktop\\chromedriver-win64\\chromedriver.exe"
    chrome_binary_path = "C:\\Users\\Administrator\\Desktop\\chrome-win64\\chrome.exe"
    service = Service(executable_path=chrome_driver_path)  # 请将 "path/to/chromedriver" 替换为你的 ChromeDriver 路径
    # service.path=""
    options = webdriver.ChromeOptions()
    options.binary_location = chrome_binary_path
    options.add_argument("--headless")

    browser = webdriver.Chrome(service=service, options=options)
    # browser = webdriver.Chrome()
    # browser.get('https://m.ctrip.com/webapp/cw/car/isd/List.html?apptype=ISD_C_CW_MAIN&channelid=235812')
    browser.get('https://accounts.ctrip.com/h5Login/login_ctrip?sibling=T')
    # print(browser.title)
    username_input = browser.find_element(By.NAME, 'ctripAccount')
    # 输入账号
    username_input.send_keys('130000000')

    # 定位密码输入框
    password_input = browser.find_element(By.NAME, 'ctripPassword')

    # 输入密码
    password_input.send_keys('1111')

    # 提交表单，根据具体情况可能需要点击登录按钮
    # password_input.send_keys(Keys.RETURN)

    button1 = browser.find_element(By.CSS_SELECTOR, 'button.agreement-checkbox')
    button1.click()

    login_button = browser.find_element(By.CLASS_NAME, 'g_btn_s')
    login_button.click()

    WebDriverWait(browser, 10).until(EC.url_changes(browser.current_url))
    # 提取登录后的 Cookie
    cookies = browser.get_cookies()
    cookie_dict = {}
    for cookie in cookies:
        cookie_dict[cookie['name']] = cookie['value']

    print("登录后的 Cookie：", cookie_dict)

    # 假设你已经提取了登录后的 Cookie，并存储在 cookie_dict 中

    # 添加 Cookie
    for name, value in cookie_dict.items():
        browser.add_cookie({'name': name, 'value': value})

    # 访问另一个页面
    browser.get('https://m.ctrip.com/webapp/cw/car/isd/List.html?apptype=ISD_C_CW_MAIN&channelid=235812')
    print(browser.title)
    # 根据类名查找所有符合条件的元素
    elements = browser.find_elements(By.CLASS_NAME, 'rn-text')

    # 遍历元素并提取文本内容
    for element in elements:
        text_content = element.text
        print("捕捉到的文本内容：", text_content)

    # elements = browser.find_elements_by_xpath('//*[@id="main"]/div[3]/div[2]/div[1]/div[2]/div')
    # for element in elements:
    #     print("多个元素的文本内容：", element.text)
    # 获取 div:nth-child(2)
    div_2 = browser.find_element_by_css_selector('#main > div.rn-flex.rn-view > div.rn-flex.rn-view > div.rn-scroller-vert.rn-view > div:nth-child(2)')
    print("div:nth-child(2) 的文本内容：", div_2.text)

    # 获取 div:nth-child(4) 及其 6 到 最后一个
    div_4_and_next = browser.find_elements_by_css_selector( '#main > div.rn-flex.rn-view > div.rn-flex.rn-view > div.rn-scroller-vert.rn-view > div:nth-child(n+4):nth-child(2n+4)')
    for div in div_4_and_next:
        print("div:nth-child(4) 及其 6 到 最后一个的文本内容：", div.text)

    browser.quit()

```