---
title: SpringBoot整合swagger2
date: 2018-10-05 22:56:59
tags: [SpringBoot,swagger]
---


参考:
>1.[Swagger使用指南](https://blog.csdn.net/sanyaoxu_2/article/details/80555328)  
>2.[Spring Boot中使用Swagger2构建强大的RESTful API文档](http://blog.didispace.com/springbootswagger2/)  
>3.[springfox-swagger原理解析与使用过程中遇到的坑](https://blog.csdn.net/w4hechuan2009/article/details/68892718)  
>4.[SwaggerAPI注解详解,以及注解常用参数配置](https://blog.csdn.net/java_yes/article/details/79183804)

# 1.认识Swagger

 作用:  
1. 接口的文档在线自动生成。  
2. 功能测试。

<!--more-->

```
Swagger是一组开源项目，其中主要要项目如下：  
     1.Swagger-tools:提供各种与Swagger进行集成和交互的工具。例如模式检验、Swagger 1.2文档转换成Swagger 2.0文档等功能。
  
     2.Swagger-core: 用于Java/Scala的的Swagger实现。与JAX-RS(Jersey、Resteasy、CXF...)、Servlets和Play框架进行集成。
 
     3.Swagger-js: 用于JavaScript的Swagger实现。
     
     4.Swagger-node-express: Swagger模块，用于node.js的Express web应用框架。
     
     5.Swagger-ui：一个无依赖的HTML、JS和CSS集合，可以为Swagger兼容API动态生成优雅文档。    
     
     6.Swagger-codegen：一个模板驱动引擎，通过分析用户Swagger资源声明以各种语言生成客户端代码。
```

# 2.导入pom.xml

版本号请根据实际情况自行更改。

```
<dependency>    
    <groupId>io.springfox</groupId>    
    <artifactId>springfox-swagger2</artifactId>    
    <version>2.2.2</version>
</dependency>

<dependency>    
	<groupId>io.springfox</groupId>   
 	<artifactId>springfox-swagger-ui</artifactId>   
 	<version>2.2.2</version>
</dependency>

```

---

# 3.创建Swagger2配置类

在Application.java同级创建Swagger2的配置类Swagger2

```
package com.swaggerTest;
 
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;
 
/**
 * Swagger2配置类
 * 在与spring boot集成时，放在与Application.java同级的目录下。
 * 通过@Configuration注解，让Spring来加载该类配置。
 * 再通过@EnableSwagger2注解来启用Swagger2。
 */
@Configuration
@EnableSwagger2
public class Swagger2 {
    
    /**
     * 创建API应用
     * apiInfo() 增加API相关信息
     * 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现，
     * 本例采用指定扫描的包路径来定义指定要建立API的目录。
     * 
     * @return
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                // 选择哪些路径和API会生成document
                .select()
                //监控指定路径
                //.apis(RequestHandlerSelectors.basePackage("com.swaggerTest.controller"))
                //對所有api進行監控
                .apis(RequestHandlerSelectors.any())
                // 对所有路径进行监控
                .paths(PathSelectors.any())
                .build();
    }
    
    /**
     * 创建该API的基本信息（这些基本信息会展现在文档页面中）
     * 访问地址：http://项目实际地址/swagger-ui.html
     * @return
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("更多请关注http://www.baidu.com")
                .termsOfServiceUrl("http://www.baidu.com")
                .contact("AsummerCat")
                .version("1.0")
                .build();
    }
}


```

访问路径:http://localhost:8080/swagger-ui.html

---

# 4.Swagger使用的注解及其说明


```
@Api：用在类上，说明该类的作用。

@ApiOperation：注解来给API增加方法说明。

@ApiImplicitParams : 用在方法上包含一组参数说明。

@ApiImplicitParam：用来注解来给方法入参增加说明。

@ApiResponses：用于表示一组响应

@ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息    
	l   code：数字，例如400    
	l   message：信息，例如"请求参数没填好"    
	l   response：抛出异常的类
   
@ApiModel：描述一个Model的信息（一般用在请求参数无法使用@ApiImplicitParam注解进行描述的时候）    
	l   @ApiModelProperty：描述一个model的属性

```

注意：@ApiImplicitParam的参数说明：
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
	<tr>
		<td>
			<p><span style="color:#FF0000;"><strong>paramType</strong></span>：指定参数放在哪个地方</p>
		</td>
		<td>
			<p>header：请求参数放置于Request Header，使用@RequestHeader获取</p><p>query：请求参数放置于请求地址，使用@RequestParam获取</p>
		<p>path：（用于restful接口）--&gt;请求参数的获取：@PathVariable</p>
		<p>body：（不常用）</p>
		<p>form（不常用）</p>
	  </td>
  </tr>
  <tr>
  	  <td>
  	  		<p>name：参数名</p>
  	  </td>
  	  <td>
  	  		<p>&nbsp;</p>	
  	  </td>
  </tr>
  <tr>
  	  <td>
  	  		<p>dataType：参数类型</p>
  	  </td>
  	  <td>
  	  		<p>&nbsp;</p>
  	  </td>
  </tr>
  <tr>
  	  <td>
  	  		<p>required：参数是否必须传</p>
  	  </td>
  	  <td>
  	  		<p>true | false</p>
  	  </td>
 </tr>
 <tr>
 	  <td>
 	  	   <p>value：说明参数的意思</p>
 	  </td>
 	  <td>
 	  		<p>&nbsp;</p>
 	  </td>
 </tr>
 <tr>
 		<td>
 			<p>defaultValue：参数的默认值</p>
 		</td>
 		<td>
 		<p>&nbsp;</p>
 		</td>
 </tr>
 </tbody>
 </table>

<font color="red">例子：</font>

```
package com.linjingc.springbootdemo.controller;

import com.linjingc.springbootdemo.User;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.*;

/**
 * 一个用来测试swagger注解的控制器
 * 注意@ApiImplicitParam的使用会影响程序运行，如果使用不当可能造成控制器收不到消息
 *
 * @author 夏天的猫
 */
@Controller
@RequestMapping("/user")
@Api(tags = "用户测试接口")
public class UserController {

    @ResponseBody
    @RequestMapping(value = "/getUserName", method = RequestMethod.GET)
    @ApiOperation(value = "根据用户编号获取用户姓名", notes = "xxx这里就是显示接口的内容")
    @ApiImplicitParam(paramType = "query", name = "userNumber", value = "用户编号", required = true, dataType = "Integer")
    public String getUserName(@RequestParam Integer userNumber) {
        if (userNumber == 1) {
            return "张三丰";
        } else if (userNumber == 2) {
            return "慕容复";
        } else {
            return "未知";
        }
    }

    @ResponseBody
    @RequestMapping(value = "/updatePassword",method = RequestMethod.POST)
    @ApiOperation(value = "修改用户密码", notes = "根据用户id修改密码")
    @ApiImplicitParams({
            @ApiImplicitParam(paramType = "query", name = "userId", value = "用户ID", required = true, dataType = "Integer"),
            @ApiImplicitParam(paramType = "query", name = "password", value = "旧密码", required = true, dataType = "String"),
            @ApiImplicitParam(paramType = "query", name = "newPassword", value = "新密码", required = true, dataType = "String")
    })
    public String updatePassword(@RequestParam(value = "userId") Integer userId, @RequestParam(value = "password") String password,
                                 @RequestParam(value = "newPassword") String newPassword) {
        if (userId <= 0 || userId > 2) {
            return "未知的用户";
        }
        if (StringUtils.isEmpty(password) || StringUtils.isEmpty(newPassword)) {
            return "密码不能为空";
        }
        if (password.equals(newPassword)) {
            return "新旧密码不能相同";
        }
        return "密码修改成功!";
    }

    @ApiOperation(value="创建用户", notes="根据User对象创建用户")
    @ApiImplicitParam(name = "user", value = "用户详细实体user", required = true, dataType = "User")
    @RequestMapping(value="", method=RequestMethod.POST)
    public String postUser(@RequestBody User user) {

        return "success";
    }
}


```
![](/img/2018-10-6/Swagger3.png)  
![](/img/2018-10-6/Swagger1.png)  
![](/img/2018-10-6/Swagger2.png)  


---

# 5.接收对象传参

```
package com.linjingc.springbootdemo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

@Data
@ApiModel(value = "用户对象")
public class User {
    @ApiModelProperty(value = "用户ID", name = "userId")  
    private Integer userId;

    @ApiModelProperty(value = "用户姓名", name = "userName")
    private String userName;

    @ApiModelProperty(value = "用户密码", name = "password")
    private String password;

    @ApiModelProperty(value = "用户手机号", name = "phone")
    private String phone;

}

```

![](/img/2018-10-6/Swagger5.png)  
![](/img/2018-10-6/Swagger4.png)  


---

# 6.定义返回值
**@ApiResponse:**  
用于方法，描述操作的可能响应。

**@ApiResponses：**  
用于方法，一个允许多个ApiResponse对象列表的包装器。 
例：

```
@ApiResponses(value = { 
            @ApiResponse(code = 2001, message = "2001:因输入数据问题导致的报错"),
            @ApiResponse(code = 403, message = "403:没有权限"),
            @ApiResponse(code = 2500, message = "2500:通用报错（包括数据、逻辑、外键关联等，不区分错误类型）")})

```
