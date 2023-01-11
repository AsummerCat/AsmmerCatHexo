---
title: 使用google的libPhone验证手机号
date: 2023-01-11 11:13:18
tags: [libPhone,google]
---
## demo地址
```
https://github.com/AsummerCat/phoneDemo.git
```

## 引入依赖
```
        <!--google验证手机号的依赖-->
        <dependency>
            <groupId>com.googlecode.libphonenumber</groupId>
            <artifactId>libphonenumber</artifactId>
            <version>8.13.2</version>
        </dependency>
```
<!--more-->
## 使用工具类
```
package com.linjingc.phonedemo;

import com.google.i18n.phonenumbers.PhoneNumberUtil;
import com.google.i18n.phonenumbers.Phonenumber;

/**
 * @author cxc
 */
public class LibPhoneNumberUtil {
    //    CountryCodeToRegionCodeMap 区号维护在这里
    private static final String DEFAULT_COUNTRY_ISO = "86";

    private static final PhoneNumberUtil PHONE_NUMBER_UTIL = PhoneNumberUtil.getInstance();

    /**
     * @param phoneNumber 手机号
     * @param areaCode    手机区号
     * @Description 手机校验逻辑
     */
    public static boolean doValid(String areaCode, String phoneNumber) {
        int code = Integer.parseInt(areaCode);
        long phone = Long.parseLong(phoneNumber);
        Phonenumber.PhoneNumber pn = new Phonenumber.PhoneNumber();
        pn.setCountryCode(code);
        pn.setNationalNumber(phone);
        return PHONE_NUMBER_UTIL.isValidNumber(pn);
    }


    /**
     * 验证国内手机号
     */
    public static boolean doValid(String phoneNumber) {
        return doValid(DEFAULT_COUNTRY_ISO, phoneNumber);
    }
}
```

## 使用
```
        boolean check = LibPhoneNumberUtil.doValid("13003808787");
         System.out.println(check?"校验成功":"校验失败");
```