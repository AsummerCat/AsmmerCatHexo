---
title: mongodb笔记之常用命令(三)
date: 2020-10-21 09:18:53
tags: [mongodb笔记]
---

# 常用命令

## 创建数据库
```
use testdb
```

## 创建集合
```
db.t_member.insert({name:"zhaomin",age:23})
```

## 查询
```
db.t_member.find()
db.t_member.findOne()
```
<!--more-->

## 修改
```
＃不会影响其他属性列 ，主键冲突会报错
db.t_member.update({name:"zhaomin"},{$set:{age:18}})
＃第三个参数为 true 则执行 insertOrUpdate 操作，查询出则更新，没查出则插入，
或者
db.t_member.update({name:"zhaomin"},{$set:{age:18}},true)
```

## 删除
```
＃删除满足条件的第一条 只删除数据 不删除索引
db.t_member.remove({age:1})
＃删除集合
db.t_member.drop();
＃删除数据库
db.dropDatabase();

```

## 查看集合
```
show collections
```

## 查看数据库
```
show dbs
```

## 插入数据
```
db.t_member.insert() ＃不允许键值重复
db.t_member.save() ＃若键值重复，可改为插入操作

```

## 批量更新
```
db.t_member.update({name:"zhaomin"},{$set:{name:"zhanmin11"}},false,true);
```

# 常用命令:
## 更新器使用$set
```
指定一个键值对，若存在就进行修改，不存在则添加
```
## $inc 数值类型加减：
只使用于数字类型，可以为指定键值对的数字类型进行加减操作
```
db.t_member.update({name:"zhangsan"},{$inc:{age:2}})

执行结果是名字叫“zhangsan”的年龄加了 2
```

## $unset : 删除指定的键
```
db.t_member.update({name:"zhangsan"},{$unset:{age:1}})
```

## 其他
$push : 数组键操作：  
1、如果存在指定的数组，则为其添加值；  
2、如果不存在指定的数组，则创建数组键，并添加值；  
3、如果指定的键不为数组类型，则报错；  

$addToSet : 当指定的数组中有这个值时，不插入，反之插入  
```
＃则不会添加到数组里
db.t_member.update({name:"zhangsan"},{$addToSet:{classes:"English"}});
```
$pop：删除指定数组的值，当 value=1 删除最后一个值，当 value=-1 删除第一个值
```
＃删除了最后一个值
db.t_member.update({name:"zhangsan"},{$pop:{classes:1}})
```

$pull : 删除指定数组指定的值
```
＃$pullAll 批量删除指定数组
db.persons.update({name:"zhangsan"},{$pull:{classes:"Chinese"}})
```
```
＃若数组中有多个 Chinese，则全删除
db.t_member.update({name:"zhangsan"},{$pull:{classes:["Chinese"]}})
```
$ : 修改指定数组时，若数组有多个对象，但只想修改其中一些，则需要定位器：
```
db.t_member.update({"classes.type":"AA"},{$set:{"classes.$.sex":"male"}})
```
$addToSet 与 $each 结合完成批量数组更新操作
```
db.t_member.update({name:"zhangsan"},{$set:{classes:{$each:["chinese"
,"art"]}}})
```
## runCommand 函数和 findAndModify 函数
```
runCommand({
findAndModify:"persons",
query:{查询器},
sort:{排序},
update:{修改器},
new:true 是否返回修改后的数据
});
```
runCommand 函数可执行 mongdb 中的特殊函数  
findAndModify 就是特殊函数之一，用于返回执行返回 update 或 remove 后的文档  
```
例如:

db.runCommand({
findAndModify:"persons",
query:{name:"zhangsan"},
update:{$set:{name:"lisi"}},
new:true
})
```

# 高级查询
## 基本查询
```
db.t_member.find({},{_id:0,name:1})

第一个空括号表示查询全部数据，第二个括号中值为 0 表示不返回，值为 1 表示返回， 
默认情况下若不指定主键，主键总是会被返回；


db.persons.find({条件},{指定键});
比较操作符：$lt: < $lte: <= $gt: > $gte: >= $ne: !=
```

## 查询条件
```
＃查询年龄大于等于 25 小于等于 27 的人
db.t_member.find({age:{$gte:25,$lte:27}},{_id:0,name:1,age:1})
＃查询出所有国籍不是韩国的人的数学成绩
db.t_member.find({country:{$ne:"韩国"}},{_id:0,name:1,country:1})
```
## 包含与不包含（仅针对于数组）
$in 或 $nin
```
＃查询国籍是中国或美国的学生信息
db.t_member.find({country:{$in:["China","USA"]}},{_id:0,name:1:countr
y:1})
```
## $or 查询
```
＃查询语文成绩大于 85 或者英语大于 90 的学生信息
db.t_member.find({$or:[{c:{$gt:85}},{e:{$gt:90}}]},{_id:0,name:1,c:1,e:1})
＃把中国国籍的学生上增加新的键 sex
db.t_member.update({country:"China"},{$set:{sex:"m"}},false,true)
＃查询出 sex 为 null 的人
db.t_member.find({sex:{$in:[null]}},{_id:0,name:1,sex:1})
```
## 正则表达式
```
＃查询出名字中存在”li”的学生的信息
db.t_member.find({name:/li/i},{_id:0,name:1})
```

## $not 的使用
$not 和$nin 的区别是$not 可以用在任何地方儿$nin 是用到集合上的
```
＃查询出名字中不存在”li”的学生的信息
db.t_member.find({name:{$not:/li/i}},{_id:0,name:1})
```

## $all 与 index 的使用
```
＃查询喜欢看 MONGOD 和 JS 的学生
db.t_member.find({books:{$all:["JS","MONGODB"]}},{_id:0,name:1})
＃查询第二本书是 JAVA 的学习信息
db.t_member.find({"books.1":"JAVA"},{_id:0,name:1,books:1})
```
## $size 的使用，不能与比较查询符同时使用
```
＃查询出喜欢的书籍数量是 4 本的学生
db.t_member.find({books:{$size:4}},{_id:0,name:1})
```
```
12.8、查询出喜欢的书籍数量大于 4 本的学生本的学生
1）增加 size 键
db.t_member.update({},{$set:{size:4}},false,true)
2）添加书籍,同时更新 size
db.t_member.update({name:"jim"},{$push:{books:"ORACL"},$inc:{size:1}
})
3）查询大于 3 本的
db.t_member.find({size:{$gt:4}},{_id:0,name:1,size:1})
12.9、$slice 操作符返回文档中指定数组的内部值
＃查询出 Jim 书架中第 2~4 本书
db.t_member.find({name:"jim"},{_id:0,name:1,books:{$slice:[1,3]}})
＃查询出最后一本书
db.t_member.find({name:"jim"},{_id:0,name:1,books:{$slice:-1}})
```

## 文档查询
```
查询出查询出在 K 上过学且成绩为 A 的学生
1）绝对查询，顺序和键个数要完全符合在 K 上过学且成绩为 A 的学生
1）绝对查询，顺序和键个数要完全符合
db.t_member.find({school:{school:"K","score":"A"}},{_id:0,name:1})

2）对象方式,但是会出错，多个条件可能会去多个对象查询
db.t_member.find({"school.school":"K","school.score":"A"},{_id:0,nam
e:1})

正确做法单条条件组查询$elemMatch
db.t_member.find({school:{$elemMatch:{school:"K",score:"A"}},{_id:0,n
ame:1})
db.t_member.find({age:{$gt:22},books:"C++",school:"K"},{_id:0,name:1,age:1,books:1,school:1})
```

## 分页与排序
```
1）limit 返回指定条数 查询出 persons 文档中前 5 条数据：
db.t_member.find({},{_id:0,name:1}).limit(5)

2）指定数据跨度 查询出 persons 文档中第 3 条数据后的 5 条数据
db.t_member.find({},{_id:0,name:1}).limit(5).skip(3)

3）sort 排序 1 为正序，-1 为倒序
db.t_member.find({},{_id:0,name:1,age:1}).limit(5).skip(3).sort({age:1})


注意:mongodb 的 key 可以存不同类型的数据排序就也有优先级
最小值->null->数字->字符串->对象/文档->数组->二进制->对象 ID->布尔->日期->时间戳->正则
->最大值
```