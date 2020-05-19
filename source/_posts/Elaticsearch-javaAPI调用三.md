---
title: Elaticsearch-javaAPI调用三
date: 2020-02-15 15:22:51
tags: [Elaticsearch,SpringBoot,java]
---

# Elaticsearch-javaAPI调用三

## demo

```
https://github.com/AsummerCat/basic_es_demo
```



# 整合spirngboot

## 导入pom

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>
```

<!--more-->

## 配置参数

```
spring:
  data:
    elasticsearch:
      repositories:
        enabled: true
      cluster-nodes: 112.74.43.136:9300
```

## 创建实体类并映射

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
@Document(indexName = "supplier_index",type = "supplier")
public class Supplier implements Serializable {
    @Id   //映射id
    private String id;
    @Field(type = FieldType.Keyword)   //映射字段，可以指定分词
    private String name;
    @Field(type = FieldType.Keyword)
    private String leader;
    @Field(type = FieldType.Keyword)
    private String phone;
    @Field(type = FieldType.Date)
    private Date create_date;
}
```

## 创建DAO

```java
public interface SupplierRepository  extends ElasticsearchRepository<Supplier,String> {
    //如果需要自定义基本的查询在这里书写
    //根据name查询
    List<Supplier> findByName(String keyWord);
    //根据name和leader查询
    List<Supplier> findByNameAndLeader(String key1,String key2);
    List<Supplier> findByNameOrLeader(String key1,String key2);
}
```

## 使用es

```java
package top.linjingc.basic_es_demo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Sort;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Date;
import java.util.Optional;

@SpringBootTest
@RunWith(SpringRunner.class)
public class TestEs {
    @Autowired
    private SupplierRepository supplierRepository;

    @Test
    public void test() {

        Supplier supplier1 = new Supplier();
        supplier1.setId("1");
        supplier1.setCreate_date(new Date());
        supplier1.setName("小明");

        //保存
        System.out.println(supplierRepository.save(supplier1));
        System.out.println(supplierRepository.findById("1"));
        //更新
        supplier1.setName("小明啊啊啊啊啊");
        System.out.println(supplierRepository.save(supplier1));
        System.out.println(supplierRepository.findById("1"));
        //删除
        supplierRepository.deleteById("1");
        System.out.println();


        //查询所有
        supplierRepository.findAll().forEach(supplier -> System.out.println(supplier));
        //根据id查询
        System.out.println(supplierRepository.findById("1016"));
        //排序查询
        supplierRepository.findAll(Sort.by(Sort.Order.asc("name"))).forEach(supplier -> System.out.println(supplier));
        //根据name查询
        System.out.println(supplierRepository.findByName("腾讯"));
        //根据name和leader查询
        System.out.println(supplierRepository.findByNameAndLeader("腾讯", "马化腾"));
        //根据name或者leader查询
        System.out.println(supplierRepository.findByNameOrLeader("0001", "马化腾"));
    }
}
```

