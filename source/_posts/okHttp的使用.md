---
title: okHttp的使用
date: 2019-11-08 11:18:07
tags: [okHttp]
---

# okHttp

[demo地址](https://github.com/AsummerCat/okhttpdemo)

[官网](https://square.github.io/okhttp/)

# 导入 pom.xml

```java
   <dependency>
                <groupId>com.squareup.okhttp3</groupId>
                <artifactId>okhttp</artifactId>
                <version>4.2.2</version>
            </dependency>
```

<!--more-->

# 基础使用

## 方式一

```java
 final MediaType JSON = MediaType.get("application/json; charset=utf-8");
			OkHttpClient client = new OkHttpClient();
			//构造builder
			Request.Builder header = new Request.Builder().url("http://www.baidu.com").header("Content-Type", "application/json");
			//构造请求体
			RequestBody data = RequestBody.create(JSON, "json数据");
			//构造请求
			Request build = header.post(data).build();

			//发送请求请求
			try (Response response = client.newCall(build).execute()) {
				System.out.println("返回结果start");
				 response.body().string();
				System.out.println("返回结果end");

			} catch (Exception e) {
			e.printStackTrace();
		}
```

## 方式2(推荐)

```java
public static void test2() {
		final MediaType JSON = MediaType.get("application/json; charset=utf-8");
		OkHttpClient client = new OkHttpClient();

		RequestBody body = RequestBody.create(JSON, "json数据");
		Request request = new Request.Builder()
				.header("Content-Type", "application/json")
				.url("http://www.baidu.com")
				.post(body)
				.build();
		//发送请求请求
		try (Response response = client.newCall(request).execute()) {
			System.out.println("返回结果start");
			response.body().string();
			System.out.println("返回结果end");

		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```