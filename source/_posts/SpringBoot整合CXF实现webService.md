---
title: SpringBoot整合CXF实现webService
date: 2019-11-07 13:57:11
tags: [SpringBoot,cxf,webService]
---

# 基于CXF实现WebService

# 导入pom.xml

```
 <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
            <version>3.3.4</version>
        </dependency>
```

<!--more-->

# 创建webService服务

## 接口

```java
/**
 * @author cxc
 * WebService接口 发布
 */
@WebService(name = "IssueService", //暴露服务名称
		targetNamespace = "http://www.webservice.cxfdemo.linjingc.com") //命名空间,一般是接口名称倒序
public interface IssueService {

	@WebMethod
	@WebResult(name = "String", targetNamespace = "")
	public String hello(@WebParam(name = "helloName") String name);
```



## 服务实现类

```java
@WebService(serviceName  = "IssueService",//与前面接口一致
		targetNamespace = "http://www.webservice.cxfdemo.linjingc.com",  //与前面接口一致
		endpointInterface = "com.linjingc.cxfdemo.webservice.IssueService")  //接口地址
@Component
public class IssueServiceImpl implements IssueService {

	@Override
	public String hello(String name) {

		return "hello World!!->"+name;
	}
}

```



##  webService服务配置

```java
@Configuration
public class WebConfig {
	@Autowired
	private Bus bus;

	@Autowired
	IssueService service;

	/*jax-ws*/
	@Bean
	public Endpoint endpoint() {
		EndpointImpl endpoint = new EndpointImpl(bus, service);
		endpoint.publish("/IssueService");
		//添加校验拦截器
		endpoint.getInInterceptors().add(new AuthInterceptor());
		return endpoint;
	}
}
```



##  webService服务用户密码拦截器配置

```java
public class AuthInterceptor extends AbstractPhaseInterceptor<SoapMessage> {
  Logger logger = LoggerFactory.getLogger(this.getClass());
  private static final String USERNAME="root";
  private static final String PASSWORD="admin";
 
  public AuthInterceptor() {
    //定义在哪个阶段进行拦截
    super(Phase.PRE_PROTOCOL);
  }
 
  @Override
  public void handleMessage(SoapMessage soapMessage) throws Fault {
    List<Header> headers = null;
    String username=null;
    String password=null;
    try {
      headers = soapMessage.getHeaders();
    } catch (Exception e) {
      logger.error("getSOAPHeader error: {}",e.getMessage(),e);
    }
 
    if (headers == null) {
      throw new Fault(new IllegalArgumentException("找不到Header，无法验证用户信息"));
    }
    //获取用户名,密码
    for (Header header : headers) {
      SoapHeader soapHeader = (SoapHeader) header;
      Element e = (Element) soapHeader.getObject();
      NodeList usernameNode = e.getElementsByTagName("username");
      NodeList pwdNode = e.getElementsByTagName("password");
       username=usernameNode.item(0).getTextContent();
       password=pwdNode.item(0).getTextContent();
      if( StringUtils.isEmpty(username)||StringUtils.isEmpty(password)){
        throw new Fault(new IllegalArgumentException("用户信息为空"));
      }
    }
    //校验用户名密码
    if(!(username.equals(USERNAME) && password.equals(PASSWORD))){
      SOAPException soapExc = new SOAPException("认证失败");
      logger.debug("用户认证信息错误");
      throw new Fault(soapExc);
    }
  }
}
```



# 创建WebService客户端调用

## webService客户端用户密码拦截器配置

```java
public class LoginInterceptor extends AbstractPhaseInterceptor<SoapMessage> {
  private String username="root";
  private String password="admin";
  public LoginInterceptor(String username, String password) {
    //设置在发送请求前阶段进行拦截
    super(Phase.PREPARE_SEND);
    this.username=username;
    this.password=password;
  }
 
  @Override
  public void handleMessage(SoapMessage soapMessage) throws Fault {
    List<Header> headers = soapMessage.getHeaders();
    Document doc = DOMUtils.createDocument();
    Element auth = doc.createElementNS("http://cxf.wolfcode.cn/","SecurityHeader");
    Element UserName = doc.createElement("username");
    Element UserPass = doc.createElement("password");
 
    UserName.setTextContent(username);
    UserPass.setTextContent(password);
 
    auth.appendChild(UserName);
    auth.appendChild(UserPass);
 
    headers.add(0, new Header(new QName("SecurityHeader"),auth));
  }
}
```



## 方式一 (需要有接口类)

```java
/**
	 * 方式1.代理类工厂的方式,需要拿到对方的接口
	 */
	public static void cl1() {
		try {
			// 接口地址
			String address = "http://localhost:8080/services/IssueService?wsdl";
			// 代理工厂
			JaxWsProxyFactoryBean jaxWsProxyFactoryBean = new JaxWsProxyFactoryBean();
			// 设置代理地址
			jaxWsProxyFactoryBean.setAddress(address);
			// 设置接口类型
			jaxWsProxyFactoryBean.setServiceClass(IssueService.class);
			//添加用户名密码拦截器
			jaxWsProxyFactoryBean.getOutInterceptors().add(new LoginInterceptor("root","admin"));
			// 创建一个代理接口实现
			IssueService cs = (IssueService) jaxWsProxyFactoryBean.create();
			// 数据准备
			String userName = "测试名成功1";
			// 调用代理接口的方法调用并返回结果
			String result = cs.hello(userName);
			System.out.println("返回结果:" + result);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```



## 方式二 (需要接口名称)

```java
/**
	 * 动态调用方式
	 */
	public static void cl2() {
		// 创建动态客户端
		JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
		Client client = dcf.createClient("http://localhost:8080/services/IssueService?wsdl");
		// 需要密码的情况需要加上用户名和密码
		 client.getOutInterceptors().add(new LoginInterceptor("root", "admin"));
		try {
			// invoke("方法名",参数1,参数2,参数3....);
			Object[] objects = client.invoke("hello", "测试名成功2");
			System.out.println("返回数据:" + objects[0]);
		} catch (java.lang.Exception e) {
			e.printStackTrace();
		}
	}
```

## 完整版本

```java
package com.linjingc.cxfdemo.consume;

import com.linjingc.cxfdemo.webservice.IssueService;
import org.apache.cxf.endpoint.Client;
import org.apache.cxf.jaxws.JaxWsProxyFactoryBean;
import org.apache.cxf.jaxws.endpoint.dynamic.JaxWsDynamicClientFactory;

public class CxfClient {
	public static void main(String[] args) {
		cl2();
	}

	/**
	 * 方式1.代理类工厂的方式,需要拿到对方的接口
	 */
	public static void cl1() {
		try {
			// 接口地址
			String address = "http://localhost:8080/services/IssueService?wsdl";
			// 代理工厂
			JaxWsProxyFactoryBean jaxWsProxyFactoryBean = new JaxWsProxyFactoryBean();
			// 设置代理地址
			jaxWsProxyFactoryBean.setAddress(address);
			// 设置接口类型
			jaxWsProxyFactoryBean.setServiceClass(IssueService.class);
			//添加用户名密码拦截器
			jaxWsProxyFactoryBean.getOutInterceptors().add(new LoginInterceptor("root","admin"));
			// 创建一个代理接口实现
			IssueService cs = (IssueService) jaxWsProxyFactoryBean.create();
			// 数据准备
			String userName = "测试名成功1";
			// 调用代理接口的方法调用并返回结果
			String result = cs.hello(userName);
			System.out.println("返回结果:" + result);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 动态调用方式
	 */
	public static void cl2() {
		// 创建动态客户端
		JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
		Client client = dcf.createClient("http://localhost:8080/services/IssueService?wsdl");
		// 需要密码的情况需要加上用户名和密码
		 client.getOutInterceptors().add(new LoginInterceptor("root", "admin"));
		try {
			// invoke("方法名",参数1,参数2,参数3....);
			Object[] objects = client.invoke("hello", "测试名成功2");
			System.out.println("返回数据:" + objects[0]);
		} catch (java.lang.Exception e) {
			e.printStackTrace();
		}
	}
}

```



这样就实现了webService的实现了