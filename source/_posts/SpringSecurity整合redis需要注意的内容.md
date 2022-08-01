---
title: SpringSecurity整合redis需要注意的内容
date: 2019-07-10 22:01:16
tags: [redis,Security]
---

# SpringSecurity整合redis需要注意的内容

SpringBoot2.x版本后 

redis默认是lettuce 做客户端

所以配置文件修改一下

```
    <dependency>
      <groupId>org.springframework.session</groupId>
      <artifactId>spring-session-data-redis</artifactId>
    </dependency>
		<dependency>
      <groupId>io.lettuce</groupId>
      <artifactId>lettuce-core</artifactId>
    </dependency>

    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-pool2</artifactId>
      <version>${commons-pool2.version}</version>
    </dependency>
    
    		<commons-pool2.version>2.6.2</commons-pool2.version>

```

 还有一个需要注意的

commons-pool2 需要导入这个包 不然会报错 缺少依赖

