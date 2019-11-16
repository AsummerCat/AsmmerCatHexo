---
title: ORCLE递归查询
date: 2019-11-15 10:14:41
tags: [数据库,orcle]
---

# orcle递归查询

树结构

## 树结构最顶级

查找树中的所有顶级父节点（辈份最长的人）。 假设这个树是个目录结构，那么第一个操作总是找出所有的顶级节点，再根据该节点找到其下属节点。

```
select * from tb_menu m where m.parent is null;
```

## 查找当前的下级

查找一个节点的直属子节点（所有儿子）。 如果查找的是直属子类节点，也是不用用到树型查询的。

```
select * from tb_menu m where m.parent=1;
```

<!--more-->

## 查找所有儿子

查找一个节点的所有直属子节点（所有后代）

```java
select * from tb_menu m start with m.id=1 connect by m.parent=prior m.id;
```

## 查找一个节点的直属父节点(父亲)

查找一个节点的直属父节点（父亲）。 如果查找的是节点的直属父节点，也是不用用到树型查询的。

```
select c.id, c.title, p.id parent_id, p.title parent_title
from tb_menu c, tb_menu p
where c.parent=p.id and c.id=6
```

