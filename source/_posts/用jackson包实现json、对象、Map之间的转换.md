---
title: 用jackson包实现json、对象、Map之间的转换
date: 2018-10-09 17:35:32
tags: [SpringBoot,java,json]
---
>可参考:  
>1.[用jackson包实现json、对象、Map之间的转换](http://rsy.iteye.com/blog/2303323)  
>2.[jackson完成json和对象/map/list互转](https://blog.csdn.net/qq_37936542/article/details/79268402)

# 导入依赖

```

<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.7.3</version>
</dependency>
```
# 转换器

`private static ObjectMapper mapper=new ObjectMapper();`


<!--more-->

---

# 编写工具类 

```
package top.linjingc.utils;

import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

/**
 * @author cxc
 */
public class JacksonTest {
    private static Logger loger = LoggerFactory.getLogger(JacksonTest.class);
    /**
     * 自定义JackSon的转换工具类
     */
    //转换器
    private static ObjectMapper mapper=new ObjectMapper();


    /**
     *Json字符串 转List
     */
    public static  List jsonToArray(Class<?> parametrized,String value) {
            List test =new ArrayList<>();
       try {
           JavaType javaType = mapper.getTypeFactory().constructParametricType(List.class, parametrized);//转换类

           System.out.println(parametrized.getClass());
           test =mapper.readValue(value, javaType);

       }catch (Exception e){
          loger.info("####################################################");
          loger.info("------List错误信息:"+e.getMessage()+"------");
           loger.info("####################################################");
       }
        return test;
    }


    /**
     *Json字符串 转Map
     */
    public static  List  jsonToMap(Class<?> parametrized,String value) {
        List test =new ArrayList();
        try {
            JavaType javaType = mapper.getTypeFactory().constructParametricType(HashMap.class,String.class, parametrized);
            test=mapper.readValue(value, javaType);

        }catch (Exception e){
            loger.info("####################################################");
            loger.info("------Map错误信息:"+e.getMessage()+"------");
            loger.info("####################################################");
        }
        return test;
    }


/**
 *任意对象转Json字符串
 * ps:Java集合 丶 基本序列化类
 *
 */
public static String objectToJson (Object test){
    String re="";
    try{
     re = mapper.writeValueAsString(test);

    }catch (Exception e){
        loger.info("####################################################");
        loger.info("------对象转换错误信息:"+e.getMessage()+"------");
        loger.info("####################################################");
    }
    return re ;
}

/**
 * Json对象转换基本类 反序列化
 */
public static Object jsonToObject (String test,Class<?> parametrized)throws Exception{
    Object re =new Object();
   try{
       re=  mapper.readValue(test, parametrized);
   }catch(Exception e){
       loger.info("####################################################");
       loger.info("------基本类转换错误信息:"+e.getMessage()+"------");
       loger.info("####################################################");
   }
   return re;
}
}

```

