---
title: oracle11G分区大表月度形式建表和查询
date: 2019-11-11 14:34:00
tags: [数据库,orcle,mysql]
---

#  oracle11G分区大表月度形式建表和查询

## 分区模式

```
根据年: INTERVAL(NUMTOYMINTERVAL(1,'YEAR'))

根据月: INTERVAL(NUMTOYMINTERVAL(1,'MONTH'))
根据天: INTERVAL(NUMTODSINTERVAL(1,'DAY'))

根据时分秒: NUMTODSINTERVAL( n, { 'DAY'|'HOUR'|'MINUTE'|'SECOND'})
```



##  1.重命名原表名称

```
alter table P_DRUG_SUI_ANALYSIS_OUT rename to P_DRUG_SUI_ANALYSIS_OUT_temp; --重命名原表名称
```



## 2.按月创建分区表

```JAVA
create table P_DRUG_SUI_ANALYSIS_OUT ---按月创建分区表
( 
	ID VARCHAR2(32) not null
	PATIENT_NO NVARCHAR2(50) not null,
	SICK_NAME NVARCHAR2(50),
	AMOUNT NUMBER,
	QUANTITY NUMBER,
	ZONE_ID NVARCHAR2(36),
	DEPART_ID NVARCHAR2(50),
	DEPARTMENT NVARCHAR2(50),
	DOC_ID NVARCHAR2(50),
	DOC_NAME NVARCHAR2(50),
	RESULT_TYPE VARCHAR2(50),
	ANALYSIS_RESULT_TYPE VARCHAR2(50),
	PRES_TIME DATE not null,
	PRES_NO NVARCHAR2(50) not null,
	ANALYSIS_TYPE VARCHAR2(50),
	DRUG_NAME NVARCHAR2(200),
	GRADE NUMBER(10)
)
PARTITION BY RANGE (PRES_TIME) INTERVAL (numtoyminterval(1, 'month'))
(partition part_t1 values less than(to_date('2019-05-01', 'yyyy-mm-dd')));
```

<!--more-->

### 解释

```
PARTITION BY RANGE (PRES_TIME) INTERVAL (numtoyminterval(1, 'month'))
(partition part_t1 values less than(to_date('2019-05-01', 'yyyy-mm-dd')));
根据PRES_TIME简历分区键   
part_t1 后面的时间 -> 表示 分区创建起始时间
比如  设置2019-05-01 就是 2019-05-01之前的为一个分片快 之后的按照月份分块
```



## 3.原表数据导入新表

```sql
insert into  P_DRUG_SUI_ANALYSIS_OUT select * from P_DRUG_SUI_ANALYSIS_OUT_temp; --原表数据导入新表
```

## 4.设置主键

```
alter table P_DRUG_SUI_ANALYSIS_OUT add constraint P_DRUG_SUI_ANALYSIS_OUT_pk_1 primary key (ID) using INDEX; --设置主键
```

如果建表的时候有设置主键 这步忽略

## 5.设置索引

给分区键 设置索引

```java
create index P_PRES_TIME_1 on P_DRUG_SUI_ANALYSIS_OUT (PRES_TIME);  --5.设置索引
```

## 6.设置允许分区表的分区键是可更新

```
alter table P_DRUG_SUI_ANALYSIS_OUT enable row movement; --设置允许分区表的分区键是可更新
```



## 7.查询当前表有多少分区

```
select table_name,partition_name,TABLESPACE_NAME from user_tab_partitions where table_name='P_DRUG_SUI_ANALYSIS_OUT';
```



## 8.查询这个表的某个（SYS_P21）里的数据 

SYS_P21 这个是分区块的名称   第七步的partition_name

```
select * from P_DRUG_SUI_ANALYSIS_OUT partition(SYS_P21);
```

