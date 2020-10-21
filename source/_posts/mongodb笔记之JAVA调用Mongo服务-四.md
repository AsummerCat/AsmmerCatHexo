---
title: mongodb笔记之JAVA调用Mongo服务(四)
date: 2020-10-21 09:19:29
tags: [mongodb笔记]
---

# 第一种使用spring-boot-starter-data-mongodb

## 添加pom依赖
```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```

## 数据源配置
```
如果mongodb端口是默认端口，并且没有设置密码，可不配置，sprinboot会开启默认的。
spring.data.mongodb.uri=mongodb://localhost:27017/springboot-db

mongodb设置了密码，这样配置：
spring.data.mongodb.uri=mongodb://name:pass@localhost:27017/dbname
```
<!--more-->

## 定义实体
```

import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.DBRef;
@Document(collection="t_customer")
public class Customer {

@Id
public String _id;
    public String carNumber;
    	@DBRef
	private Role role;

    public String get_id() {
        return _id;
    }

    public void set_id(String _id) {
        this._id = _id;
    }

    public String getCarNumber() {
        return carNumber;
    }

    public void setCarNumber(String carNumber) {
        this.carNumber = carNumber;
    }
}

```

## dao层
```
@Repository
public interface CustomerRepository extends MongoRepository<Customer, String> {

    //public Customer findByFirstName(String firstName);
   //public List<Customer> findByLastName(String lastName);
   //自定义查询
   @Query(value="{'role.$id': ?0 }")
	public User findByRoleId(ObjectId id);
}
```

## 接口调用
```
RestController
@RequestMapping("/mongo")
public class MongoController {

    @Autowired
    private KdVehicleRepository kdVehicleRepository;

    @Autowired  //使用模板
    private MongoTemplate mongoTemplate;


    @RequestMapping("/list")
    public List<Customer> dobegin() throws Exception {
       List<Customer> list =  kdVehicleRepository.findAll(); //dao查询
     //List<KdVehicle> list2 =  mongoTemplate.findAll(Customer.class);//也可以
        return list;
    }
}
```

# 第二种 使用模板调用
```
    @Autowired  //使用模板
    private MongoTemplate mongoTemplate;
```
如果要使用模板的话    
要么就是使用`spring-boot-starter-data-mongodb`的依赖 

要么就是使用`原生的mongodb连接`+`morphia`依赖



# 第三种 原生api调用
```
//创建连接
MongoClient client=MongoClient.create("mongodb//hostname:27017,hostname1:27018")

//获取数据库
MongoDatabase db=client.getDatabase("test_db");

//获取集合
MongoCollection coll=db.getCollection("t_meber");

//插入document
Document doc=new Document("name","小明")
.append("age",18);
coll.insertOne(doc);

//查询
coll.find();


```
# 第四种 缩写方式

```
//创建连接
Mongo mongo=new Mongo("127.0.0.1","27017");

//获取数据库
Db db=new Db(mongo,"test_db");

//获取集合
DbCollection coll=db.getCollection("t_meber");

//插入dbObject
DbObject dbObject=BaseDbObject();
dbObject.put("name","小明");
dbObject.put("age",18);
coll.insert(dbObject);

//查询
DbCursor cursor=coll.find();
for ( DbObject ob : cursor)
{
    System.out.print(ob);
}
```

# 第五种 使用morphia

```
final Morphia morphia=new Morphia();
//获取连接
Database ds=morphia.createDatabase(new MongoClient("127.0.0.1",27017),"db_name");

//创建自定义实体 
/**
注意实体属性里需要有这个属性才可以返回key.getId
@Id
private ObjectId id;
**/
User user=new User();
user.setName("小明");
user.setAge(18);

//保存数据
Key<User> key=ds.save(user);

//返回id
System.out.print(key.getId);

```