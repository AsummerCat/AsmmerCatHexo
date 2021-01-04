---
title: JAVA造假数据jFairy
date: 2021-01-04 17:58:54
tags: [java]
---

# 快速创建测试数据

# ![DEMO地址](https://github.com/AsummerCat/jfairy_demo)
## 方案有两种
1. java-faker
2. jFairy


### 以下部分是创建jFairy的过程

### 导入pom依赖
```
        <!--jfairy创建测试数据-->
        <dependency>
            <groupId>io.codearte.jfairy</groupId>
            <artifactId>jfairy</artifactId>
            <version>0.5.9</version>
        </dependency>

```
<!--more-->

### 用法
#### 构造函数
```
	//构建jfairy的语法本土化配置 默认英文
		Fairy fairy = Fairy.create(Locale.SIMPLIFIED_CHINESE);
```
##### 用户基本信息相关
```
Person person = fairy.person();
		System.out.println("全名:" + person.getLastName() + person.getFirstName());
		System.out.println("年龄:" + person.getAge());
		System.out.println("地址:" + person.getAddress().getAddressLine1());
		System.out.println("城市:" + person.getAddress().getCity());
		System.out.println("邮编:" + person.getAddress().getPostalCode());
		System.out.println("出生年月日:" + person.getDateOfBirth());
		System.out.println("用户名:" + person.getUsername());
		System.out.println("邮箱:" + person.getEmail());
		System.out.println("密码:" + person.getPassword());
		System.out.println("身份证:" + person.getNationalIdentityCardNumber());
		System.out.println("电话:" + person.getTelephoneNumber());

```

##### 公司相关
```
Company company = fairy.company();
		System.out.println("公司名称:" + company.getName());
		System.out.println("公司域名:" + company.getDomain());
		System.out.println("公司网址:" + company.getUrl());
		System.out.println("公司序列号:" + company.getVatIdentificationNumber());
```

##### 随机相关
```
BaseProducer baseProducer = fairy.baseProducer();
		//随机数
		System.out.println(baseProducer.randomBetween(1, 100));
		//随机正反
		System.out.println(baseProducer.trueOrFalse());
	
```

##### 产品相关
```
CreditCard creditCard = fairy.creditCard();
		//有效期
		System.out.println(creditCard.getExpiryDate());
		//有效期字符串
		System.out.println(creditCard.getExpiryDateAsString());
		//供应商
		System.out.println(creditCard.getVendor());
```

##### 随机时间相关
```
DateProducer dateProducer = fairy.dateProducer();
		//之前11年的时间
		System.out.println(dateProducer.randomDateInThePast(11));
		//未来五年
		System.out.println(dateProducer.randomDateInTheFuture(5));
		//公元2000-2020年之间
		System.out.println(dateProducer.randomDateBetweenYears(2000, 2020));
		//2000年之后到现在的随机时间
		System.out.println(dateProducer.randomDateBetweenYearAndNow(2000));
```

##### 网络相关
```
NetworkProducer networkProducer = fairy.networkProducer();
		//随机获取ip地址
		System.out.println(networkProducer.ipAddress());
		//获取https地址
		System.out.println(networkProducer.url(true));
		//获取http地址
		System.out.println(networkProducer.url(false));

```

##### 字符串相关
```
TextProducer textProducer = fairy.textProducer();
		//直接返回字符串
		System.out.println(textProducer.result("123"));
		//随机构建短语
		System.out.println(textProducer.sentence());

```



## 完整内容
```
package com.example.jfairy_demo;

import io.codearte.jfairy.Fairy;
import io.codearte.jfairy.producer.BaseProducer;
import io.codearte.jfairy.producer.DateProducer;
import io.codearte.jfairy.producer.company.Company;
import io.codearte.jfairy.producer.net.NetworkProducer;
import io.codearte.jfairy.producer.payment.CreditCard;
import io.codearte.jfairy.producer.person.Person;
import io.codearte.jfairy.producer.text.TextProducer;

import java.util.Locale;

/**
 * jfairy的测试脚本
 *
 * @author cxc
 */
public class JfairyTest {

	public static void main(String[] args) {
		//构建jfairy的语法本土化配置 默认英文
		Fairy fairy = Fairy.create(Locale.SIMPLIFIED_CHINESE);


		//用户基本信息相关
		Person person = fairy.person();
		System.out.println("全名:" + person.getLastName() + person.getFirstName());
		System.out.println("年龄:" + person.getAge());
		System.out.println("地址:" + person.getAddress().getAddressLine1());
		System.out.println("城市:" + person.getAddress().getCity());
		System.out.println("邮编:" + person.getAddress().getPostalCode());
		System.out.println("出生年月日:" + person.getDateOfBirth());
		System.out.println("用户名:" + person.getUsername());
		System.out.println("邮箱:" + person.getEmail());
		System.out.println("密码:" + person.getPassword());
		System.out.println("身份证:" + person.getNationalIdentityCardNumber());
		System.out.println("电话:" + person.getTelephoneNumber());

		System.out.println("=======================================");
		System.out.println("=======================================");


		//公司相关
		Company company = fairy.company();
		System.out.println("公司名称:" + company.getName());
		System.out.println("公司域名:" + company.getDomain());
		System.out.println("公司网址:" + company.getUrl());
		System.out.println("公司序列号:" + company.getVatIdentificationNumber());

		System.out.println("=======================================");
		System.out.println("=======================================");


		//随机相关
		BaseProducer baseProducer = fairy.baseProducer();
		//随机数
		System.out.println(baseProducer.randomBetween(1, 100));
		//随机正反
		System.out.println(baseProducer.trueOrFalse());

		System.out.println("=======================================");
		System.out.println("=======================================");


		// 产品相关
		CreditCard creditCard = fairy.creditCard();
		//有效期
		System.out.println(creditCard.getExpiryDate());
		//有效期字符串
		System.out.println(creditCard.getExpiryDateAsString());
		//供应商
		System.out.println(creditCard.getVendor());

		System.out.println("=======================================");
		System.out.println("=======================================");


		//随机时间相关
		DateProducer dateProducer = fairy.dateProducer();
		//之前11年的时间
		System.out.println(dateProducer.randomDateInThePast(11));
		//未来五年
		System.out.println(dateProducer.randomDateInTheFuture(5));
		//公元2000-2020年之间
		System.out.println(dateProducer.randomDateBetweenYears(2000, 2020));
		//2000年之后到现在的随机时间
		System.out.println(dateProducer.randomDateBetweenYearAndNow(2000));

		System.out.println("=======================================");
		System.out.println("=======================================");

		//网络相关
		NetworkProducer networkProducer = fairy.networkProducer();
		//随机获取ip地址
		System.out.println(networkProducer.ipAddress());
		//获取https地址
		System.out.println(networkProducer.url(true));
		//获取http地址
		System.out.println(networkProducer.url(false));

		System.out.println("=======================================");
		System.out.println("=======================================");

		//字符串相关
		TextProducer textProducer = fairy.textProducer();
		//直接返回字符串
		System.out.println(textProducer.result("123"));
		//随机构建短语
		System.out.println(textProducer.sentence());

	}
}


```