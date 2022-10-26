---
title: Springboot整合Neo4j
date: 2022-10-26 16:44:47
tags: [SpringBoot,Neo4j]
---
# Springboot整合Neo4j

## demo地址
https://github.com/AsummerCat/neo4jDemo.git/

## 导入依赖
```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-neo4j</artifactId>
        </dependency>
```

## 新增配置
```
spring:
  neo4j:
    uri: bolt://127.0.0.1:27687
    authentication:
      username: naco4j
      password: 123456
```
<!--more-->
## 创建对应接口类
继承`Neo4jRepository` 底层有很多初始化方法
```
package com.linjingc.neo4jdemo.dao;

import com.linjingc.neo4jdemo.entity.Person;
import org.springframework.data.neo4j.repository.Neo4jRepository;
import org.springframework.stereotype.Repository;

/**
 * 创建对应接口类 继承Neo4jRepository
 */
@Repository
public interface PersonRepository extends Neo4jRepository<Person, Long> {

}

```

## 编写Neo4j 节点实体类
//节点
`@Node("Person")`
//建立关系
` @Relationship(type = "children", direction = Relationship.Direction.OUTGOING)`

```
package com.linjingc.neo4jdemo.entity;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.neo4j.core.schema.GeneratedValue;
import org.springframework.data.neo4j.core.schema.Node;
import org.springframework.data.neo4j.core.schema.Property;
import org.springframework.data.neo4j.core.schema.Relationship;

import java.io.Serializable;
import java.util.Set;

/**
 * neo4j节点
 */
@Data
@Node("person")
public class Person implements Serializable {
    @Id
    @GeneratedValue
    private Long id;

    @Property
    private String name;

    /**
     * @Relationship这个注解表示关系，type是关系名称，direction是方向，例子中的是person——>child
     * 孩子节点
     */
    @Relationship(type = "children", direction = Relationship.Direction.OUTGOING)
    private Set<Children> childList;

    /**
     * 父节点 person<-parent
     */
    @Relationship(type = "parent", direction = Relationship.Direction.INCOMING)
    private Set<Parent> parentList;
}

```


## 测试代码
```
package com.linjingc.neo4jdemo;

import com.linjingc.neo4jdemo.dao.PersonRepository;
import com.linjingc.neo4jdemo.entity.Children;
import com.linjingc.neo4jdemo.entity.Parent;
import com.linjingc.neo4jdemo.entity.Person;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.HashSet;
import java.util.Optional;
import java.util.Set;

@SpringBootTest
class Neo4jDemoApplicationTests {

    @Autowired
    private PersonRepository personRepository;

    @Test
    void contextLoads() {
    }

    @Test
    public void test() {
        Person person = new Person();
        person.setName("小东");
        //保存节点数据
        personRepository.save(person);

        //查询节点id
        Optional<Person> byId = personRepository.findById(41L);
        Person resPerson = byId.get();
        System.out.println(resPerson);

        //删除节点
        personRepository.deleteById(21L);


        //构建关系
        Set<Parent> parentList = new HashSet<>();
        Set<Children> childList = new HashSet<>();

        Person info = new Person();
        info.setName("本人");
        Children children = new Children();
        children.setName("儿子");
        Parent parent = new Parent();
        parent.setName("爸爸");
        childList.add(children);
        parentList.add(parent);
        info.setChildList(childList);
        info.setParentList(parentList);
        personRepository.save(info);

    }

}

```


