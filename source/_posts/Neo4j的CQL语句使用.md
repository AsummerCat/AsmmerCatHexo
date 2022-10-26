---
title: Neo4j的CQL语句使用
date: 2022-10-26 16:44:15
tags: [Neo4j]
---
# Ne4oj的CQL语句使用

## CQL命令

| CQL命令 | 用法 |
| --- | --- |
| CREATE | 创建节点,关系和属性 |
| MATCH | 检索有关节点,关系和属性数据 |
| RETURN | 返回查询结果 |
| WHERE | 提供条件过滤检索数据 |
| DELETE | 删除节点和关系 |
|REMOVE|删除节点和关系的属性|
|ORDER BY|排序检索数据|
|SET|添加或更新标签|

<!--more-->

## LOAD CSV
```
LOAD CSV '{csv-dir}/test/csv' as line
CREATE(:tagName {name: line[1], year: toInteger(line[2])})
```
导入CSV文件, 并且创建节点的标签, 及其对应属性 name,year

例子:
```
LOAD CSV '{csv-dir}/test/csv' as line
CREATE(:tagName {from:line[1],relation:line[3],to:line[0]})
```

## 创建节点 CREATE
```
#创建简单节点
CREATE(n)

#创建多个节点
CREATE(n)(m)

# 创建带标签和属性的节点并返回节点
create (n:person {name:'小明'}) return n
```

## 查询MATCH
```
match (n:标签名称 {name:'xxx'}) return n.name,id(n) limit 25
```
name表示属性,则查询指定属性的数据
或者这种查询方式:
```
match (n:节点名称) where n.name='孙悟空' return n.name,id(n)
```
多表查询
```
match (n:person {name:'小明'}),(m:class) where m.form='小明' 
return n.name,m.relation,m.to
```

## 创建关系
```
match (n:person {name:'小明'}),(m:person {name:'小红'})
create (n)-[r:同学]->(m) return n.name,type(r),m.name
```
小明的同学是->小红

返回结果:

| n.name | type(r) | m.name |
| --- | --- | --- |
| 小明 | 同学 | 小红 |

## 删除 DELETE
```
# 删除节点 (前提:节点不存在关系)
MATCH(n:person{name:"小明"}) delete n

# 删除关系
MATCH(n:person{name:"小明"})<-[r]-(m) delete r return type(r)

```

## 删除 REMOVE 删除节点的属性
```
# 删除属性
MATCH(n:role{name:"fox"}) remove n.age return n

#删除标签
MATCH(n:role{name:"fox"}) remove n:role return n

```

## SET子句 修改属性
```
# 新增或者更新属性值

MATCH(n:role {name:"fox"}) set n.age=32 return n
```

## ORDER BY排序
```
MATCH (n:'xiyou1') RETURN id(n),n.name order by id(n) desc
```

## union语句
```
MATCH(n:role{name:"fox"}) return n.name
union
MATCH(n:role{name:"bis"}) return n.name

```

## 有NULL值的属性节点
```
mmatch(n:"稀有") where n.label is null return id(n),n.name,n.tail,n.label
```

## IN操作符
```
match(n:"稀有") where n.label in['小明','小东'] return id(n),n.name,n.tail,n.label
```

## 创建索引
```
# 创建索引
create index on :'稀有' (name)

# 删除索引 
drop index on:'稀有'(name)
```

## UNIQE约束
```
# 创建唯一约束
create constraint on (n:xiyou) assert n.name is unique

# 删除唯一约束
drop constraint on (n:xiyou) assert n.name is unique

```

## DISTINCT去重
```
match (n:'西游') return distinct(n)
```

