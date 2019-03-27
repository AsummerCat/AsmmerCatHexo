---
title: orcle存储过程
date: 2019-03-24 20:32:54
tags: [数据库,orcle]
---

# ORCLE存储过程

## 创建存储过程

```
--括号内的是外部传入的参数 : 参数名 IN 参数类型  

create or replace procedure PASS_GET_RANGE_OF_DIAGNOSTIC(BEGIN_TIME IN DATE,END_TIME IN DATE,DRUG_CODES IN VARCHAR2) is   

begin  

--执行主体  

end PASS_GET_RANGE_OF_DIAGNOSTIC;  

```

<!--more-->

## 声明变量

```
需要注意的是 变量声明需要在开头 给变量赋值需要在begin中

--声明变量  变量名称 变量类型
V_START_TIME VARCHAR2(100);  
V_END_TIME DATE;  


--begin之后

--给变量赋值   变量 := 赋值内容
V_START_TIME :='p_check_resi_pres_detail';  
V_END_TIME :=TO_DATE(END_TIME,'YYYY-MM');  
  
```

## 声明游标 

```
-- 游标定义在 begin之前 
-- 游标使用参数 
-- cursor 游标名称(参数 类型,....) is sql查询语句 ;

cursor location_data(BEGIN_TIME DATE,END_TIME DATE,DRUG_CODES VARCHAR2) is   
select t.id,t.main_diagnose,t.diagnose from P_CHECK_RESI_PRES t   
where (t.main_diagnose is not null or  t.diagnose is not null)  
 and t.delete_remark=0  and t.send_time>=BEGIN_TIME and t.send_time<=END_TIME    
 and t.PRES_STATUS in(1,3) ;   

```

### 遍历游标

```
for l in location_data(BEGIN_TIME,END_TIME,DRUG_CODES) loop  --遍历游标   
      BEGIN  
     
-- 循环主体 l为记录行 可以l.属性名  

      end;  
    end loop;   
```

### 输出控制台

```
dbms_output.put_line('输出数据'||l.id||'--->'||info.diagnose);  
```

# 自定义表名的游标循环

## 声明变量

```
--声明变量  

TYPE ref_cursor_type IS REF CURSOR; --定义一个动态游标  
users ref_cursor_type;   --定义游标类型  
v_id P_CHECK_RESI_PRES.id%type; --列名赋值  这里用来返回值  
v_main_diagnose P_CHECK_RESI_PRES.main_diagnose%type;  
v_diagnose P_CHECK_RESI_PRES.diagnose%type;  
V_SQL   VARCHAR2(2000) ;  --SQL文本  

```

## sql拼接

```
V_SQL := 
'select t.id,t.main_diagnose,t.diagnose from '|| MIAN_TABLE || ' t   
inner join '||DETAIL_TABLE||' p on p.parent_id =t.id   
and p.drug_code in (  
SELECT REGEXP_SUBSTR ('||DRUG_CODES||', ''[^,]+'', 1,rownum) as drugcodes from dual    
connect by rownum<=LENGTH ('||DRUG_CODES||') - LENGTH (regexp_replace('||DRUG_CODES||', '','', ''''))+1
)   
left join p_his_medicine j on j.physic_code=p.drug_code   
where t.delete_remark=0 and t.PRES_STATUS in(1,3)' ;   

--打印SQL语句
dbms_output.put_line(V_SQL);    

```

## 手动打开游标

```
-- 这里标识
OPEN users FOR V_SQL;  --打开游标
```

## 执行方法体 fetch

```
LOOP  
 fetch data into v_id,v_main_diagnose,v_diagnose; --遍历游标 顺便查询语句赋值给 into  
   exit when data%notfound;  --判断结果不为空  
   
--判断属性是否为空  
IF info.diagnose is not null THEN  
  --具体执行方法  
END IF;  

 end loop; 

```

## 手动关闭游标

```
CLOSE data;  --关闭游标   
```