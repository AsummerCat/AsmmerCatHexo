---
title: 关于oracle存储过程的相关操作
date: 2020-04-05 19:55:25
tags: [数据库,oracle,存储过程]
---

# 关于oracle存储过程的相关操作
## 创建包


```
create or replace package sp_test as
  procedure sp_test1(v_str varchar2);
  procedure sp_test2;
  procedure sp_test3;
end sp_test;
```
## 创建包体

<!--more-->

```
create or replace package body sp_test as
  procedure sp_test1(v_str varchar2) is 
  begin 
  dbms_output.put_lint(v_str);
  end;
    procedure sp_test2() is 
  begin 
  null;
  end;
.....
end sp_test;


## 抛出异常
rasie ....

```

## 异常捕获

```
--这个是在头部定义的捕获参数
e_pool_null     exception;

    if paramList.count() = 0 then
      ls_msg := '医嘱申请记录为空';
      --抛出异常
      raise e_parm_notvalid;
    end if;
    
    --捕获异常处理
 exception
    when e_parm_notvalid then
      dbms_output.put_line('异常抛出-1');
      return - 1;
      return 0;
```
系统自带异常

```
 exception  --异常捕捉，不要把有需要的代码放在异常捕捉后面，有异常才会执行异常代码下所有代码，没有异常不会执行  
             when errorException then  
                  errorCode := SQLCODE;  
                  errorMsg := SUBSTR(SQLERRM, 1, 200);   
                  flag := 'false';  
                  out_return := 'flag=' || flag || ',errorCode=' || errorCode || ',errorMsg=' || errorMsg;  
             when others then  
                   errorCode := SQLCODE;      
                   errorMsg := SUBSTR(SQLERRM, 1, 200);   
                   flag := 'false';  
                   out_return := 'flag=' || flag || ',errorCode=' || errorCode || ',errorMsg=' || errorMsg;  
           
```


## 自定义对象类型

```
type type_pres_apply_records is record(
    data_rowid             rowid,
    pres_apply_no          ZOEPRES.PRES_APPLY_RECORDS.PRES_APPLY_NO%type,
    pres_no                ZOEPRES.PRES_APPLY_RECORDS.pres_no%type,
    pres_sub_no            ZOEPRES.PRES_APPLY_RECORDS.pres_sub_no%type,
    exec_time              ZOEPRES.PRES_APPLY_RECORDS.exec_time%type,
    freq_name              ZOEPRES.PRES_APPLY_RECORDS.freq_name%type,
    untuck_freq_flag       ZOEPRES.PRES_APPLY_RECORDS.untuck_freq_flag%type,
    times                  ZOEPRES.PRES_APPLY_RECORDS.times%type,
    pres_apply_status      ZOEPRES.PRES_APPLY_RECORDS.pres_apply_status%type,
    freq_sub_no            ZOEPRES.PRES_APPLY_RECORDS.freq_sub_no%type,
    event_no               ZOEPRES.PRES_APPLY_RECORDS.event_no%type,
    patient_id             ZOEPRES.PRES_APPLY_RECORDS.patient_id%type,
    lay_drug_flag          ZOEPRES.PRES_APPLY_RECORDS.lay_drug_flag%type,
    usage_drug_code        ZOEPRES.PRES_APPLY_RECORDS.usage_drug_code%type,
    sub_no                 ZOEPRES.PRES_APPLY_RECORDS.sub_no%type,
    actual_duty_group_code ZOEPRES.PRES_APPLY_RECORDS.actual_duty_group_code%type,
    lay_apply_no           ZOEPRES.PRES_APPLY_RECORDS.lay_apply_no%type,
    lay_sub_no             ZOEPRES.PRES_APPLY_RECORDS.lay_sub_no%type,
    oneself_drug_flag      ZOEPRES.PRES_APPLY_RECORDS.oneself_drug_flag%type,
    apply_operator_code    ZOEPRES.PRES_APPLY_RECORDS.apply_operator_code%type,
    apply_time             ZOEPRES.PRES_APPLY_RECORDS.apply_time%type,
    created_time           ZOEPRES.PRES_APPLY_RECORDS.created_time%type,
    memo                   ZOEPRES.PRES_APPLY_RECORDS.memo%type);
```

## 自定义数组
### 基础类型数组

```
定长数组
type String_list IS VARRAY (99) of varchar2(50);
  type int_list IS VARRAY (99) OF NUMBER; 
  
  
可变数组
type arr is table of NUMBER index by binary_integer; --可变数组
```
### 自定义数组

```
--标记自定义可变集合
  type pres_apply_records_pipeline is table of type_pres_apply_records index by binary_integer;

```

## 存储过程和函数的差别
存储过程有out 没有返回值
函数有out,也有return;

```
 --存储过程
  procedure execApplys(paramList in pres_apply_records_pipeline,
                       rs_msg    out varchar2,
                       rs_code   out varchar2);

  --函数
  function test(paramList in pres_apply_records_pipeline,
                rs_msg    out varchar2) return integer;
```






## 打印输出

```
dbms_output.put_line('sp_test2');
```

## if判断

```
一、单个IF
1、

if a=...  then
.........
end if;

2、

if a=... then
......
else
....
end if;

二、多个IF

if a=..  then
......
elsif a=..  then
....
end if;     
这里中间是“ELSIF”，而不是ELSE IF 。这里需要特别注意

```




## for循环

```
for i in 1 .. paramList.count loop
    
      dbms_output.put_line(paramList(i).pres_apply_no);
    
    end loop;
```
还有一种游标的方式

## 游标

```
--创建游标 并且输入语句
--数据类型 可以是数组
now_pool_info type_pres_apply_records;
     type c_type is ref cursor;
     c1 c_type;
open c1 for V_SQL;
 loop   
   --将结果赋值给now_pool_info对象
    fetch c1 into now_pool_info;
    EXIT WHEN c1%NOTFOUND OR c1%NOTFOUND IS NULL;
    dbms_output.put_line(now_pool_info.pres_apply_no);
 end loop;
 close c1;
```

## 执行动态sql语句

```
/*** DDL ***/  
create or replace procedure CREATE_TABLE(CREATE_SQL VARCHAR2) is
begin
  EXECUTE IMMEDIATE CREATE_SQL; -- 'create table temp_table(table_name varchar2(50))'
end CREATE_TABLE;


/*** DML ***/  
declare   
   v_1 varchar2(8);   
   v_2 varchar2(10);   
   str varchar2(50);   
begin   
   v_1:='测试';   --这里的v_1,v_2可以是直接存储过程中传过来的参数
   v_2:='北京';   
   str := 'INSERT INTO test (name ,address) VALUES (:1, :2)';   
   EXECUTE IMMEDIATE str USING v_1, v_2;   
   commit;   
end;
```
## 执行动态sql (推荐)

```
select xxxx into 映射你要的数据结构 from 表
```

## 开启独立事务
比如嵌套事务


```
function test3(operator in varchar2) return number
   is
   --开启独立事务
   PRAGMA AUTONOMOUS_TRANSACTION;
    begin
     INSERT INTO zoecomm.COM_I18N_DICT_INFO (I18N_ITEM_CODE) VALUES ('B');
        
      return 1;  
    end;
```

## 在存储过程传入String 切割成数组

```
type mytype is table of VARCHAR2(4000) index by binary_integer;


function my_split(piv_str in varchar2, piv_delimiter in varchar2)
  --piv_str 为字符串，piv_delimiter 为分隔符
  return mytype is
  j        int := 0;
  i        int := 1;
  len      int := 0;
  len1     int := 0;
  str      varchar2(4000);
  tt mytype;
begin
  len  := length(piv_str);
  len1 := length(piv_delimiter);
  while j < len loop
    j := instr(piv_str, piv_delimiter, i);
    if j = 0 then
      j   := len;
      str := substr(piv_str, i);
      
      tt(tt.count+1) := str;
      if i >= len then
        exit;
      end if;
    else
      str := substr(piv_str, i, j - i);
      i   := j + len1;
      tt(tt.count+1) := str;
    end if;
  end loop;
  return tt;
end my_split;
```
## 查询并且判空

```
    begin
      select trim(param_value)
        into rs_parm_value
        from zoedict.dic_sys_param
       where param_english_name = as_parm_name;
  exception
      when NO_DATA_FOUND then
        raise e_sys_parm_notfound;
    end;
    
```


## 游标查询防止空游标运行下一步流程

```
EXIT WHEN c1%NOTFOUND OR c1%NOTFOUND IS NULL;
```


## 判断字符串出现次数->以目标字符开头

```
instr('源字符串' , '目标字符串' ,'开始位置','第几次出现')

返回的是 目标字符的下标
```

## 查询语句直接赋值到数组
bulk collect into 数组

```
 begin
        select *
          bulk collect into SHARE_RECORD_INFO
          from dual;
```

## 手动抛出异常 外层捕获

```
code取值访问 错误号的范围是-20,000到-20,999
raise_application_error(-20001,error_msg);

接收异常
rs_msg  := SUBSTR(SQLERRM, 1, 512);
```

## oracle的包中常量定义


```
ls_brbs CONSTANT varchar2(6):='30949';
```

## oracle使用表结构的定义当做对象

```
inpLayDrugPool ZOEAPPLY.APP_INP_LAY_DRUG_POOL%rowtype;

参数名 表名%rowtype;
```

## oracle使用表结构定义的对象插入数据

```
insert into ZOEAPPLY.APP_INP_LAY_DRUG_POOL values 定义的对象;
```

## 计算数值 取整这类操作

```
select trunc(4.136,2) as 直接丢弃尾巴,
round(4.136,2) as 四舍五入,
ceil(4.136) as 向上取整,
floor(4.136) as   向下整位,
to_char(11.0010,'fm9999990.09999') as test
from dual
```

## sql执行获取被修改数量

```
  SELECT *
          into ide_data
          from ZOEPRES.PRES_APPLY_RECORDS_POOL a
         where a.PRES_APPLY_NO = nowPools(i).pres_apply_no
           and a.PRES_APPLY_STATUS = 0
           for update;
        P_ROWS := SQL%ROWCOUNT;
```

## oracle将数组存入到游标中

```
定义游标
 type type_cursor_records is ref cursor;
 
 存入数组:
 open paramLis for select * from table (paramList1);
```

## 发送http请求

```
 url := 'http://localhost/test.jsp';  
 req := utl_http.begin_request(url);
resp := utl_http.get_response(req);
LOOP
utl_http.read_line(resp, value, TRUE);
dbms_output.put_line('网站回复' || value);
END LOOP;
utl_http.end_response(resp);
EXCEPTION
WHEN utl_http.end_of_body THEN
utl_http.end_response(resp);
END;
```

