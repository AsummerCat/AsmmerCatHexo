---
title: javaApi调用mongodb的使用
date: 2019-11-22 14:25:44
tags: [java,mongodb]
---

# 插入

##  普通版本

```java
@Test
  public void testInsert(){

      MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

      // 获取指定数据库
      MongoDatabase database = mongoClient.getDatabase("test");

      // 获取指定集合
      MongoCollection<Document> collection = database.getCollection("users");

      // 创建需要插入的单个文档
      Document doc = new Document("name", "MongoDB")
              .append("type", "database")
              .append("count", 1)
              .append("versions", Arrays.asList("v3.2", "v3.0", "v2.6"))
              .append("info", new Document("x", 203).append("y", 102));

      // 测试插入
      collection.insertOne(doc);

      // 测试插入多个文档
      ArrayList<Document> documents = new ArrayList<Document>();
      for (int i = 0; i < 100; i++) {
           documents.add(new Document("i",i));
      }

      collection.insertMany(documents);
      mongoClient.close();
  }
```

<!--more-->

## springboot

```java
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
	 * @param users
	 * @return
	 */
	public List<User> batchSave(List<User> users) {
		List<User> data = mongoTemplate.insert(users);
		return data;
	}
```



# 更新

## 普通版本

```java
 @Test
  public void test3(){
      MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

      // 获取指定数据库
      MongoDatabase database = mongoClient.getDatabase("test");

      // 获取指定集合
      MongoCollection<Document> collection = database.getCollection("users");

      // update t_user set i = 110 where i = 10
      //UpdateResult updateResult = collection.updateOne(new Document("i", 10), new Document("$set", new Document("i", 110)));

      // update t_user set i =10 where i < 20
      // lt(key,value)  静态导入  import static com.mongodb.client.model.Filters.lt;
      collection.updateMany(lt("i",20),new Document("$set",new Document("i",10).append("test","test value")));

      mongoClient.close();
  }

```

# springboot

```
/**
	 * 更新对象
	 */
	public void updateUser(User user) {
		Query query = new Query(Criteria.where("id").is(user.getId()));
		
		
		Query query1 = new Query(Criteria.where("id").lt(user.getId()).and("id").in("1"));
		
		Update update = new Update().set("age", user.getAge()).set("name", user.getName());
		//更新查询返回结果集的第一条
		UpdateResult updateResult = mongoTemplate.updateFirst(query, update, User.class);
		//更新查询返回结果集的所有
//		   mongoTemplate.updateMulti(query, update, User.class);
	}
```



# 删除

## 普通版本

```java
@Test
  public void test4(){
      MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

      // 获取指定数据库
      MongoDatabase database = mongoClient.getDatabase("test");

      // 获取指定集合
      MongoCollection<Document> collection = database.getCollection("users");

      DeleteResult deleteResult = null;

      //deleteResult =collection.deleteOne(eq("i", 110));

      deleteResult = collection.deleteMany(lt("i", 20));

      System.out.println("删除的文档数："+deleteResult.getDeletedCount());

      mongoClient.close();
  }
```

## springboot

```java
/**
	 * 删除对象
	 */
	public void deleteUser(User user) {
		Query query = new Query(Criteria.where("id").is(user.getId()));
		DeleteResult remove = mongoTemplate.remove(query, User.class);
//		mongoTemplate.remove(user);
	}
```

# 查询

## 普通版本

```java
@Test
  public void test5(){
      MongoClient mongoClient = new MongoClient("192.168.128.156", 27017);

      // 获取指定数据库
      MongoDatabase database = mongoClient.getDatabase("test");

      // 获取指定集合
      MongoCollection<Document> collection = database.getCollection("users");

      // 查所有
      FindIterable<Document> documents = null;

      documents = collection.find();

      // 条件查询 select * fromt t_user where i = 20
      documents = collection.find(eq("i",20));

      // select * from t_user where i <= 20
      documents = collection.find(lte("i",30));

      // select * from t_user where i = 30 or i = 40 or i = 50
      documents = collection.find(in("i",30,40,50));

      // SELECT * FROM t_user WHERE i = 50 OR i < 25
      documents = collection.find(or(new Document("i",50),lt("i",25)));

      // 模糊查询 参数二：模糊条件
      documents = collection.find(regex("test","test"));

      // 排序 SELECT * FROM t_user WHERE i >= 90 order by i desc
      documents = collection.find(gt("i",90)).sort(Sorts.descending("i"));

      // 分页查询
      documents = collection.find().skip(50).limit(10);

      for (Document document : documents) {
          System.out.println(document);
      }

      mongoClient.close();
  }

```

## springboot

### 根据字段查询

```
/**
	 * 查找名称
	 */
	public User findUserByName(String name) {
		Query query = new Query(Criteria.where("name").is(name));
		User one = mongoTemplate.findOne(query, User.class);
		return one;
	}
	
	
	
```

### 模糊查询

```
/**
	 * 模糊查询
	 * @param search
	 * @return
	 */
	public List<User> findByLikes(String search){
		Query query = new Query();

//完全匹配
	//	Pattern pattern1 = Pattern.compile("^张$", Pattern.CASE_INSENSITIVE);
//右匹配
	//	Pattern pattern2 = Pattern.compile("^.*张$", Pattern.CASE_INSENSITIVE);
//左匹配
		//Pattern pattern3 = Pattern.compile("^张.*$", Pattern.CASE_INSENSITIVE);
//模糊匹配
	//	Pattern pattern4 = Pattern.compile("^.*张.*$", Pattern.CASE_INSENSITIVE);

		Pattern pattern = Pattern.compile("^.*" + search + ".*$" , Pattern.CASE_INSENSITIVE);
		Criteria criteria = Criteria.where("name").regex(pattern);
		query.addCriteria(criteria);
		List<User> lists = mongoTemplate.findAllAndRemove(query, User.class);
		return lists;
	}

```

### 排序查询

Sort sort = Sort.by(Direction.ASC, "age");

```java
	/**
	 * 排序查询
	 */
	public List<User> sortData(){
		Sort sort = Sort.by(Direction.ASC, "age");
//		Criteria criteria = Criteria.where("address").is(0);//查询条件
//		Query query = new Query(criteria);
		List<User> users = mongoTemplate.find(new Query().with(sort).limit(1000), User.class);
		return users;
	}

```

### 获取指定条数   limit

```java
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
```

### 分页查询

Pageable pageable = PageRequest.of(page, size);

```java
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
```



### 分组查询

Aggregation agg = Aggregation.newAggregation();

```java
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
```



###  去重查询

`mongoTemplate.findDistinct(query,"age",User.class, Integer.class);`

```
/**
	 * 去重查询
	 * @return
	 */
	public List<Integer>  getDistinctList(){

		Query query = new Query();
		List<Integer> list = mongoTemplate.findDistinct(query,"age",User.class, Integer.class);
    	return list;
    }
```



# 原子性操作

## 查询并更新

```java
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
```

## 查询并删除

```java
/**
	 * 原子性操作 查询删除
	 */
	public User atomicityFindAndRemove(){
		Query query = new Query(Criteria.where("name").is("小明41"));
		User andModify = mongoTemplate.findAndRemove(query, User.class);
		return andModify;
	}

```



