---
title: java爬虫-chromedriver获取版本
date: 2024-05-04 16:01:48
tags: [java, 爬虫,chromedriver]
---
这里的动作是根据本地浏览器下载对应的chromedriver版本,为后续的爬虫做准备
```
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>4.16.1</version>
        </dependency>

        <dependency>
            <groupId>io.github.bonigarcia</groupId>
            <artifactId>webdrivermanager</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
```
<!--more-->

```
package com.example.easypoidemo;

import static org.junit.Assert.*;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.openqa.selenium.NoSuchElementException;

public class BaiduTestDemo {
WebDriver driver;

    @BeforeClass
    public static void setUpBeforeClass() throws Exception {
        //设置chromedirver，默认下载路径为C:\Users\Administrator\.cache\selenium\chromedriver
        //是当前登录用户的\.cache\selenium\chromedriver路径

        WebDriverManager.chromedriver().setup();

    }

    @AfterClass
    public static void tearDownAfterClass() throws Exception {
    }

    @Test
    public void test() {
        driver = new ChromeDriver();
        String url = "http://www.baidu.com";
        driver.get(url);
        driver.manage().window().maximize();
        driver.findElement(By.id("kw")).sendKeys("selenium");
        driver.findElement(By.id("su")).click();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }

        assertTrue(isElementPresent(By.partialLinkText("selenium"))); //验证是否有selenium 相关搜索链接

        driver.quit();
    }


    private boolean isElementPresent(By by) {
        try {
            driver.findElement(by);
            return true;
        } catch (NoSuchElementException e) {
            return false;

        }

    }

}

```
