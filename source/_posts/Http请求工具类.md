---
title: Http请求工具类
date: 2019-11-19 14:03:26
tags: [java]
---

# Http请求工具类

## 导入pom.xml

```
 <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.10</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.62</version>
        </dependency>
```

<!--more-->

## 工具类

```
package com.linjingc.httpclientdemo.util;

import com.alibaba.fastjson.JSONObject;
import lombok.extern.log4j.Log4j2;
import org.apache.http.HttpStatus;
import org.apache.http.NameValuePair;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.config.Registry;
import org.apache.http.config.RegistryBuilder;
import org.apache.http.conn.socket.ConnectionSocketFactory;
import org.apache.http.conn.socket.PlainConnectionSocketFactory;
import org.apache.http.conn.ssl.SSLConnectionSocketFactory;
import org.apache.http.conn.ssl.TrustSelfSignedStrategy;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.DefaultHttpRequestRetryHandler;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.ssl.SSLContextBuilder;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.security.KeyManagementException;
import java.security.KeyStoreException;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Http请求工具类
 * HttpClient
 *
 * @author cxc
 * @date 2019年11月19日14:01:00
 */
@Log4j2
public class HttpUtils {
	private static String POST_SENT = "http://localhost:8070/sent/postMethod";
	private static String POST_SENT_JSON = "http://localhost:8070/sent/postMethodJson";
	private static String GET_SENT = "http://localhost:8070/sent/getMethod";


	// utf-8字符编码
	public static final String CHARSET_UTF_8 = "utf-8";
	/**
	 * HTTP内容类型。相当于form表单的形式，提交数据
	 */
	public static final String CONTENT_TYPE_FORM_URL = "application/x-www-form-urlencoded;charset=utf-8";
	/**
	 * HTTP内容类型。相当于form表单的形式，提交数据
	 */
	public static final String CONTENT_TYPE_JSON_URL = "application/json;charset=utf-8";

	/**
	 * 连接管理器
	 */
	private static PoolingHttpClientConnectionManager pool;
	/**
	 * 请求配置
	 */
	private static RequestConfig requestConfig;

	/**
	 * 初始化HttpClient
	 */
	static {
		try {
			log.info("初始化HttpClientTest--->start");
			SSLContextBuilder builder = new SSLContextBuilder();
			builder.loadTrustMaterial(null, new TrustSelfSignedStrategy());
			SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
					builder.build());
			// 配置同时支持 HTTP 和 HTPPS
			Registry<ConnectionSocketFactory> socketFactoryRegistry = RegistryBuilder.<ConnectionSocketFactory>create().register(
					"http", PlainConnectionSocketFactory.getSocketFactory()).register(
					"https", sslsf).build();
			// 初始化连接管理器
			pool = new PoolingHttpClientConnectionManager(socketFactoryRegistry);
			// 将最大连接数增加到200，实际项目最好从配置文件中读取这个值
			pool.setMaxTotal(200);
			// 设置最大路由
			pool.setDefaultMaxPerRoute(2);
			// 根据默认超时限制初始化requestConfig
			int socketTimeout = 10000;
			int connectTimeout = 10000;
			int connectionRequestTimeout = 10000;
			requestConfig = RequestConfig.custom().setConnectionRequestTimeout(
					connectionRequestTimeout).setSocketTimeout(socketTimeout).setConnectTimeout(
					connectTimeout).build();
			log.info("初始化HttpClientTest---->end");
		} catch (NoSuchAlgorithmException | KeyStoreException | KeyManagementException e) {
			e.printStackTrace();
		}
		// 设置请求和传输超时时间
		requestConfig = RequestConfig.custom().setSocketTimeout(50000).setConnectTimeout(50000)
				.setConnectionRequestTimeout(50000).build();
		// 设置请求和传输超时时间
		//RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(2000).setConnectTimeout(2000).build();
	}


	private static CloseableHttpClient getHttpClient() {

		CloseableHttpClient httpClient = HttpClients.custom()
				// 设置连接池管理
				.setConnectionManager(pool)
				// 设置请求配置
				.setDefaultRequestConfig(requestConfig)
				// 设置重试次数
				.setRetryHandler(new DefaultHttpRequestRetryHandler(0, false))
				.build();
		return httpClient;
	}

	public static void main(String[] args) {
//		postTest();
//		postJsonTest();
		getTest();
//		post(POST_SENT);
//		get("http://www.baidu.com");
	}

	private static void postJsonTest() {
		JSONObject text = JSONObject.parseObject("{\"age\":105,\"id\":\"小小小\"}");
		String result1 = HttpUtils.post(POST_SENT_JSON, text);
		System.out.println(result1);
	}

	private static void postTest() {
		Map<String, Object> params = new HashMap<>(16);
		params.put("name", "小明");
		params.put("age", "16");
		String result = HttpUtils.post(POST_SENT, params);
		System.out.println(result);
	}

	private static void getTest() {
		Map<String, Object> params = new HashMap<>(16);
		params.put("name", "小明");
		params.put("age", "16");
		String result = HttpUtils.get(GET_SENT, params);
		System.out.println(result);
	}


	/**
	 * Http发送json请求  post
	 *
	 * @param url   链接地址
	 * @param param 参数
	 * @return
	 */
	public static String post(String url, JSONObject param) {
		return doPostToJson(url, param, null);
	}

	/**
	 * Http发送json请求   post
	 *
	 * @param url           链接地址
	 * @param param         参数
	 * @param requestConfig 设置请求和传输超时时间
	 * @return
	 */
	public static String post(String url, JSONObject param, RequestConfig requestConfig) {
		return doPostToJson(url, param, requestConfig);
	}

	/**
	 * Http发送普通表单请求   post
	 *
	 * @param url    链接地址
	 * @param params 参数
	 * @return
	 */
	public static String post(String url, Map<String, Object> params) {
		return doPost(url, params, null);
	}

	/**
	 * Http发送普通表单请求 post
	 *
	 * @param url           链接地址
	 * @param params        参数
	 * @param requestConfig 设置请求和传输超时时间
	 * @return
	 */
	public static String post(String url, Map<String, Object> params, RequestConfig requestConfig) {
		return doPost(url, params, requestConfig);
	}

	/**
	 * Http发送普通表单请求 post 无参
	 *
	 * @param url
	 * @return
	 */
	public static String post(String url) {
		return doPost(url, new HashMap<>(2), requestConfig);
	}


	/**
	 * Http发送普通表单请求 get
	 *
	 * @param url    链接地址
	 * @param params 参数
	 * @return
	 */
	public static String get(String url, Map<String, Object> params) {
		return doGet(url, params, null);
	}

	/**
	 * Http发送普通表单请求 get
	 *
	 * @param url           链接地址
	 * @param params        参数
	 * @param requestConfig 设置请求和传输超时时间
	 * @return
	 */
	public static String get(String url, Map<String, Object> params, RequestConfig requestConfig) {
		return doGet(url, params, requestConfig);
	}

	/**
	 * Http发送普通表单请求 get 无参
	 *
	 * @param url 链接地址
	 * @return
	 */
	public static String get(String url) {
		return doGet(url, new HashMap<>(2), requestConfig);
	}


	/**
	 * get请求，参数拼接在地址上
	 *
	 * @param url 请求地址加参数
	 * @return 响应
	 */
	private static String doGet(String url, RequestConfig requestConfig) {
		//返回值
		String result = null;

		// 获取httpClient
		CloseableHttpClient httpclient = getHttpClient();

		CloseableHttpResponse response = null;
		try {
			//创建Get请求
			HttpGet httpGet = new HttpGet(url);
			httpGet.setConfig(requestConfig);
			//表单消息头
			httpGet.addHeader("Content-Type", CONTENT_TYPE_FORM_URL);
			//发起请求
			response = httpclient.execute(httpGet);
			/**请求发送成功，并得到响应**/
			result = resultData(response);

		} catch (Exception e) {
			log.error("请求失败--->" + url, e);
		} finally {
			try {
				if (response != null) {
					EntityUtils.consume(response.getEntity());
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return result;
	}

	/**
	 * get请求，参数拼接在地址上
	 *
	 * @param url 请求地址加参数
	 * @return 响应
	 */
	private static String doGet(String url, Map<String, Object> params, RequestConfig requestConfig) {
		//返回值
		String result = null;

		// 获取httpClient
		CloseableHttpClient httpclient = getHttpClient();

		CloseableHttpResponse response = null;
		try {
			//参数集合
			URIBuilder builder = new URIBuilder(url);
			// 提交参数发送请求
			if (params != null && !params.isEmpty()) {
				List<NameValuePair> parameForToken = new ArrayList<>();
				//发起post请求所需要的参数，如果有多个，就add多个
				for (Map.Entry<String, Object> entry : params.entrySet()) {
					//发起post请求所需要的参数，如果有多个，就add多
					parameForToken.add(new BasicNameValuePair(entry.getKey(), String.valueOf(entry.getValue())));
				}
				//封装作用传递参数    需要指定UTF-8
				builder.setParameters(parameForToken);
			}
			//创建Get请求
			HttpGet httpGet = new HttpGet(builder.build());
			httpGet.setConfig(requestConfig);

			//表单消息头
			httpGet.addHeader("Content-Type", CONTENT_TYPE_FORM_URL);
			//发起请求
			response = httpclient.execute(httpGet);
			/**请求发送成功，并得到响应**/
			result = resultData(response);

		} catch (Exception e) {
			log.error("请求失败--->" + url, e);
		} finally {
			try {
				if (response != null) {
					EntityUtils.consume(response.getEntity());
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return result;
	}


	/**
	 * Http发送json请求
	 *
	 * @param url           链接地址
	 * @param param         参数
	 * @param requestConfig 设置请求和传输超时时间
	 * @return
	 */
	private static String doPostToJson(String url, JSONObject param, RequestConfig requestConfig) {
		//返回值
		String result = null;
		// 获取httpClient
//		CloseableHttpClient httpclient = HttpClients.createDefault();
		CloseableHttpClient httpclient = getHttpClient();

		CloseableHttpResponse response = null;
		try {
			//创建post请求
			HttpPost httpPost = new HttpPost(url);
			httpPost.setConfig(requestConfig);
			// 提交参数发送请求
			if (param != null && !param.isEmpty()) {
				StringEntity data = new StringEntity(param.toString(), CHARSET_UTF_8);
				// post方法中，加入json数据
				httpPost.setEntity(data);
			}
			//http请求消息头
			httpPost.addHeader("Content-Type", CONTENT_TYPE_JSON_URL);
			//发起请求
			response = httpclient.execute(httpPost);
			/**请求发送成功，并得到响应**/
			result = resultData(response);

		} catch (Exception e) {
			log.error("请求失败--->" + url, e);
		} finally {
			try {
				if (response != null) {
					EntityUtils.consume(response.getEntity());
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return result;
	}

	/**
	 * Http发送普通表单请求
	 *
	 * @param url           链接地址
	 * @param params        参数
	 * @param requestConfig 设置请求和传输超时时间
	 * @return
	 */
	private static String doPost(String url, Map<String, Object> params, RequestConfig requestConfig) {
		//返回值
		String result = null;

		// 获取httpClient
		CloseableHttpClient httpclient = getHttpClient();
		CloseableHttpResponse response = null;
		try {
			//创建post请求
			HttpPost httpPost = new HttpPost(url);
			httpPost.setConfig(requestConfig);
			// 提交参数发送请求
			if (params != null && !params.isEmpty()) {
				List<NameValuePair> parameForToken = new ArrayList<>();
				//发起post请求所需要的参数，如果有多个，就add多个
				for (Map.Entry<String, Object> entry : params.entrySet()) {
					//发起post请求所需要的参数，如果有多个，就add多
					parameForToken.add(new BasicNameValuePair(entry.getKey(), String.valueOf(entry.getValue())));
				}
				//封装FORM表单实体对象,作用传递参数    需要指定UTF-8
				UrlEncodedFormEntity urlEncodedFormEntity = new UrlEncodedFormEntity(parameForToken, "utf-8");
				httpPost.setEntity(urlEncodedFormEntity);
			}
			//表单消息头
			httpPost.addHeader("Content-Type", CONTENT_TYPE_FORM_URL);
			//发起请求
			response = httpclient.execute(httpPost);
			/**请求发送成功，并得到响应**/
			result = resultData(response);

		} catch (Exception e) {
			log.error("请求失败--->" + url, e);
		} finally {
			try {
				if (response != null) {
					EntityUtils.consume(response.getEntity());
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return result;
	}


	/**
	 * 请求成功返回响应
	 *
	 * @param response
	 * @return
	 * @throws IOException
	 */
	private static String resultData(CloseableHttpResponse response) throws IOException {
		String result;
		/**请求发送成功，并得到响应**/
		if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
			/**读取服务器返回过来的json字符串数据**/
			result = EntityUtils.toString(response.getEntity());
			return result;
		} else {
			throw new RuntimeException("请求失败返回状态码--->" + response.getStatusLine().getStatusCode());
		}
	}
}

```

