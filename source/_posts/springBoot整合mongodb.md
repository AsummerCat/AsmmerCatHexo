---
title: springBoot整合mongodb
date: 2019-11-22 14:03:45
tags: [SpringBoot,mongodb]
---

# SpringBoot整合mongodb

## [demo地址](https://github.com/AsummerCat/mongdb-demo)

## 导入pom.xml	

```java
       <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        
                  <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>fastjson</artifactId>
           <version>1.2.62</version>
       </dependency>
```

## 配置文件

添加数据库配置（application.yml）

```java
spring:
  data:
    mongodb:
      authentication-database: admin
      username: admin
      password: password
      port: 27017
      database: 111
      host: localhost

```

## 使用

```java
@Resource
    private MongoTemplate mongoTemplate;
```

<!--more-->

## 创建实体

```
package com.linjingc.mongdbdemo.entity;

import lombok.Data;

@Data
public class User {
	private String id;
	private String name;
	private int age;
	private String address;
	private String uuuu;
}

```



## dao层

```java
package com.linjingc.mongdbdemo.dao;


import com.alibaba.fastjson.JSONObject;
import com.linjingc.mongdbdemo.entity.User;
import com.mongodb.client.result.DeleteResult;
import com.mongodb.client.result.UpdateResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.aggregation.Aggregation;
import org.springframework.data.mongodb.core.aggregation.AggregationResults;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.regex.Pattern;

import static org.springframework.data.domain.Sort.Direction;

@Component
public class MongoDao {
	@Autowired
	private MongoTemplate mongoTemplate;


	/**
	 * 保存
	 */
	public User save(User user) {
		User resultData = mongoTemplate.save(user);
		return resultData;
	}

	/**
	 * 批量插入
	 *
	 * @param users
	 * @return
	 */
	public List<User> batchSave(List<User> users) {
		List<User> data = mongoTemplate.insert(users);
		return data;
	}


	/**
	 * 查找名称
	 */
	public User findUserByName(String name) {
		Query query = new Query(Criteria.where("name").is(name));
		User one = mongoTemplate.findOne(query, User.class);
		return one;
	}

	/**
	 * 更新对象
	 */
	@Transactional(rollbackFor = {Exception.class})
	public void updateUserAddress(User user) {
		Query query = new Query(Criteria.where("name").is(user.getName()));


//		Query query1 = new Query(Criteria.where("id").lt(user.getId()).and("id").in("1"));

		Update update = new Update().set("address", "第四次");
		//更新查询返回结果集的第一条
		UpdateResult updateResult = mongoTemplate.updateFirst(query, update, User.class);
		updateResult.getMatchedCount();
		//更新查询返回结果集的所有
//		   mongoTemplate.updateMulti(query, update, User.class);
	}


	/**
	 * 删除对象
	 */
	public void deleteUser(User user) {
		Query query = new Query(Criteria.where("name").is(user.getName()));
		DeleteResult remove = mongoTemplate.remove(query, User.class);
//		mongoTemplate.remove(user);
	}

	/**
	 * 模糊查询
	 *
	 * @param search
	 * @return
	 */
	public List<User> findByLikes(String search) {
		Query query = new Query();

//完全匹配
		//	Pattern pattern1 = Pattern.compile("^张$", Pattern.CASE_INSENSITIVE);
//右匹配
		//	Pattern pattern2 = Pattern.compile("^.*张$", Pattern.CASE_INSENSITIVE);
//左匹配
		//Pattern pattern3 = Pattern.compile("^张.*$", Pattern.CASE_INSENSITIVE);
//模糊匹配
		//	Pattern pattern4 = Pattern.compile("^.*张.*$", Pattern.CASE_INSENSITIVE);

		Pattern pattern = Pattern.compile("^.*" + search + ".*$", Pattern.CASE_INSENSITIVE);
		Criteria criteria = Criteria.where("name").regex(pattern);
		query.addCriteria(criteria);
		List<User> lists = mongoTemplate.findAllAndRemove(query, User.class);
		return lists;
	}


	/**
	 * 排序查询
	 */
	public List<User> sortData() {
		Sort sort = Sort.by(Direction.ASC, "age");
//		Criteria criteria = Criteria.where("address").is(0);//查询条件
//		Query query = new Query(criteria);
		List<User> users = mongoTemplate.find(new Query().with(sort), User.class);
		return users;
	}

	/**
	 * 获取指定条数
	 */
	public List<User> limitData() {
		Sort sort = Sort.by(Direction.ASC, "age");
//		Criteria criteria = Criteria.where("address").is(0);//查询条件
//		Query query = new Query(criteria);
		List<User> users = mongoTemplate.find(new Query().limit(3), User.class);
		return users;
	}

	/**
	 * 分页查询
	 * @return
	 */
	public List<User> pageData(){
		//页数从0开始
		int page=0;
		int size=2;
	Query query = new Query();
	Pageable pageable = PageRequest.of(page, size);
	List<User> resolveRules = mongoTemplate.find(query.with(pageable), User.class);

	return resolveRules;
}


	/**
	 * 分组查询
	 */
    public List<JSONObject> groupByData(){
	    Aggregation agg = Aggregation.newAggregation(

			    // 第一步：挑选所需的字段，类似select *，*所代表的字段内容
			    Aggregation.project("name", "age", "address"),
			    // 第二步：sql where 语句筛选符合条件的记录
//            Aggregation.match(Criteria.where("userId").is(map.get("userId"))),
			    // 第三步：分组条件，设置分组字段
			    Aggregation.group("age")
					    .avg("age").as("allAge")
					    .max("name").as("DataName"),

			    // 第四部：排序（根据某字段排序 倒序）
			    Aggregation.sort(Sort.Direction.DESC,"age"),
			    // 第五步：数量(分页)
			    Aggregation.limit(15),
			    // 第六步：重新挑选字段  显示的字段
			    Aggregation.project("allAge","DataName")

	    );

	    AggregationResults<JSONObject> results = mongoTemplate.aggregate(agg, User.class, JSONObject.class);

	    List<JSONObject> mappedResults = results.getMappedResults();

    	return mappedResults;
    }


	/**
	 * 去重查询
	 * @return
	 */
	public List<Integer>  getDistinctList(){

		Query query = new Query();
		List<Integer> list = mongoTemplate.findDistinct(query,"age",User.class, Integer.class);
    	return list;
    }


	/**
	 * 原子性操作 查询更新
	 */
	public User atomicityFindAndUpdate(){
		Query query = new Query(Criteria.where("name").is("小明41"));
		Update update = new Update().set("address", "原子性操作成功");
		User andModify = mongoTemplate.findAndModify(query, update, User.class);
		//返回的是原值
		return andModify;
	}

	/**
	 * 原子性操作 查询删除
	 */
	public User atomicityFindAndRemove(){
		Query query = new Query(Criteria.where("name").is("小明41"));
		User andModify = mongoTemplate.findAndRemove(query, User.class);
		return andModify;
	}


}

```



## controller层

```java
package com.linjingc.mongdbdemo.controller;

import com.alibaba.fastjson.JSONObject;
import com.linjingc.mongdbdemo.dao.MongoDao;
import com.linjingc.mongdbdemo.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;
import java.util.Random;

/**
 * MongDb测试类
 */
@RestController
@RequestMapping("test")
public class MongoDbController {

	@Autowired
	private MongoDao mongoDao;

	@RequestMapping("add")
	public String add() {
		User user = new User();
		user.setId(String.valueOf(System.currentTimeMillis()));
//		user.setAddress("小明的地址");
		Random rand = new Random();
		user.setAge(rand.nextInt(100));
		user.setName("小明" + rand.nextInt(100));
		user.setUuuu("1111");
		mongoDao.save(user);
		return "插入成功---->" + user.toString();
	}


	@RequestMapping("update")
	public String update(int num) {
		User user = new User();
		user.setId(String.valueOf(System.currentTimeMillis()));
		user.setAddress("小明的更新的地址");
		user.setName("小明" + num);
		mongoDao.updateUserAddress(user);
		return "更新成功---->";
	}

	@RequestMapping("find")
	public String find(int num) {
		User user = new User();
		user.setName("小明" + num);
		User userByName = mongoDao.findUserByName(user.getName());
		return "查找成功---->" + userByName.toString();
	}

	@RequestMapping("delete")
	public String delete(int num) {
		User user = new User();
		user.setName("小明" + num);
		mongoDao.deleteUser(user);
		return "删除成功---->";
	}


	/**
	 * 排序查询成功
	 *
	 * @return
	 */
	@RequestMapping("sortData")
	public String sortData() {
		List<User> users = mongoDao.sortData();
		return "排序查询成功---->" + users.toString();
	}


	/**
	 * 排序查询成功
	 *
	 * @return
	 */
	@RequestMapping("limitData")
	public String limitData() {
		List<User> users = mongoDao.limitData();
		return "获取指定条数查询成功---->" + users.toString();
	}


	/**
	 * 分页查询
	 *
	 * @return
	 */
	@RequestMapping("pageData")
	public String pageData() {
		List<User> users = mongoDao.pageData();
		return "分页查询成功---->" + users.toString();
	}

	/**
	 * 分组查询
	 *
	 * @return
	 */
	@RequestMapping("groupByData")
	public String groupByData() {
		List<JSONObject> users = mongoDao.groupByData();
		return "分组查询成功---->" + users.toString();
	}

	/**
	 * 去重查询
	 *
	 * @return
	 */
	@RequestMapping("getDistinctList")
	public String getDistinctList() {
		List<Integer> users = mongoDao.getDistinctList();
		return "分组查询成功---->" + users.toString();
	}

	/**
	 * 原子性操作 查询更新
	 * @return
	 */
	@RequestMapping("atomicityFindAndUpdate")
	public String atomicityFindAndUpdate() {
		User users = mongoDao.atomicityFindAndUpdate();
		return "原子性操作查询更新成功---->" + users.toString();
	}


	/**
	 * 原子性操作 查询删除
	 * @return
	 */
	@RequestMapping("atomicityFindAndRemove")
	public String atomicityFindAndRemove() {
		User users = mongoDao.atomicityFindAndRemove();
		return "原子性操作查询删除成功---->" + users.toString();
	}


}


```

