---
title: SpringBoot参数校验
date: 2022-02-06 21:51:23
tags: [SpringBoot]
---

# springboot参数校验

## 导入相关依赖
```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>
```
<!--more-->

## 校验工具类
```
package com.syswin.systoon.edusyncmq.utils;

import com.alibaba.fastjson.JSON;
import org.apache.commons.collections.CollectionUtils;

import javax.validation.ConstraintViolation;
import javax.validation.Validation;
import javax.validation.Validator;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class ValidateUtil {
    private static Validator validator = Validation.buildDefaultValidatorFactory()
            .getValidator();

    public static void beanValidate(Object obj) {
        Map<String, String> validatedMsg = new HashMap<>();
        Set<ConstraintViolation<Object>> constraintViolations = validator.validate(obj);
        for (ConstraintViolation<Object> c : constraintViolations) {
            validatedMsg.put(c.getPropertyPath().toString(), c.getMessage());
        }
        if (CollectionUtils.isNotEmpty(constraintViolations)) {
            throw new RuntimeException(JSON.toJSONString(validatedMsg));
        }

    }

}

```

## 使用
```
 ValidateUtil.beanValidate(syncAllDataReq);
```
