---
title: java操作oracle返回存储过程带参数传入和返回
date: 2020-04-09 14:37:08
tags: [java,oracle,存储过程,mybatis]
---

# java操作oracle返回存储过程带参数传入和返回

## 这边需要注意的一点是

```
如果传入自定义数组的话 最好新建一个
CREATE OR REPLACE TYPE "TYPE_APPLY_RECORD"  AS OBJECT(
   APPLY_NO       NUMBER(20),
   ITEM_CODE      VARCHAR2(400)
 )

CREATE OR REPLACE TYPE "TYPE_APPLY_RECORD_PIPELINE"
                                      AS TABLE OF TYPE_APPLY_RECORD





不要使用包里的
定义类型
type type_pres_apply_records is record();
 自定义数组
  type pres_apply_records_pipeline is table of type_pres_apply_records index by binary_integer;

```

<!--more-->

## jdbcTemplate方式

<!--more-->

```
List resultList = (List) jdbcTemplate.execute(
			        con -> {
				        // region 1.转换ARRAY
				        // 2.设置后台包入参
				        if (con.isWrapperFor(OracleConnection.class)) {
					        con = con.unwrap(OracleConnection.class);
				        }
				        //自定义数组的转换 传入参数
				        ARRAY array = getArray(applys, con);
                        //你的执行存储过程
				        String storedProc = "{call zoepackage.ZOE_APPLYS.test(?,?,?,? )}";
				        CallableStatement cs = con.prepareCall(storedProc);
				        //设置传参 带下标
				        cs.setArray(1, array);
				        cs.setString(2, operator);
				        cs.registerOutParameter(3, Types.VARCHAR);
				        //此处为游标类型
				        cs.registerOutParameter(4, OracleTypes.CURSOR);


				        return cs;
			        }, (CallableStatementCallback) cs -> {
				        // 3.调用,获取返回值
				        List<Map<String, Object>> resultsMap = new ArrayList<>();
				        cs.execute();
				        Map<String, Object> rowMap = new HashMap<>();
				        //基本类型接收
				        rowMap.put("valtest", cs.getString(3) );
                        //游标类型接收
				        ResultSet rs=(ResultSet) cs.getObject(4);
				        while(rs.next()){
				        //这边的话获取到记录 根据下标去找到你需要的字段
					        int anInt = rs.getInt(1);
				        }


				        resultsMap.add(rowMap);
				        cs.close();
				        return resultsMap;
			        });
```
### 定义自定义数组

```
private ARRAY getArray(List<PresApplyRecords> listData, Connection con) throws SQLException {
        //传入的记录
		STRUCT[] struts = new STRUCT[listData.size()];
		//遍历将oracle数组的位置填满
		for (int i = 0; i < listData.size(); i++) {

			PresApplyRecords presApplyRecords = listData.get(i);
			Object[] result = {presApplyRecords.getPresApplyNo(), presApplyRecords.getPresNo(), presApplyRecords.getPresSubNo()
					
			};
			//获取自定义数组的单条记录的结构类型
			//注意大写
			StructDescriptor st = new StructDescriptor("ZOEPACKAGE.ZOE_APPLYS.TYPE_PRES_APPLY_RECORDS".toUpperCase(), con);
			struts[i] = new STRUCT(st, con, result);
		}
		//获取自定义数组
		ArrayDescriptor arrayDept = ArrayDescriptor.createDescriptor("ZOEPACKAGE.ZOE_APPLYS.PRES_APPLY_RECORDS_PIPELINE".toUpperCase(), con);
		//转换为oracle的数据结构
		ARRAY deptArrayObject = new ARRAY(arrayDept, con, struts);
		return deptArrayObject;
	}
```



## mybatis

### 如果传入数组需要建立一个BaseTypeHandler进行类型转换

```
package com.zoe.optimus.service.config;




import com.zoe.optimus.service.prescribe.entity.prescribe.PresApplyRecords;
import oracle.jdbc.OracleConnection;
import oracle.sql.*;
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**
 * 数据库自定义类型 数组->List 对照关系
 *
 * @author cxc
 * @apiNote 用来创建存储过程自定义数组类型和java映射关系
 * @date 2020年4月7日17:00:37
 */
//这个注解声明了他是处理jdbc类型的  @MappedTypes()也可以用这个注解指定javaType.进行约束,一般不用
@MappedJdbcTypes(JdbcType.ARRAY)
public class PresApplyRecordsArrayTypeHandler extends BaseTypeHandler {
	/**
	 * 赋值部分 发送参数 入参
	 * @param preparedStatement
	 * @param i
	 * @param o
	 * @param jdbcType
	 * @throws SQLException
	 */
	@Override
	public void setNonNullParameter(PreparedStatement preparedStatement, int i, Object o, JdbcType jdbcType) throws SQLException {
		   //获取连接池 并且这里是进行druid连接池转换oracle
//		try (Connection conn = preparedStatement.getConnection().unwrap(OracleConnection.class)) {
		  //注意了不能关闭当前连接
			Connection conn = preparedStatement.getConnection().unwrap(OracleConnection.class);
			Array array;
			try{
			List<PresApplyRecords> list =(List<PresApplyRecords>) o;
			array = getArray(conn, "ZOEPACKAGE.TYPE_APPLY_RECORD", "ZOEPACKAGE.TYPE_APPLY_RECORD_PIPELINE", list);
			preparedStatement.setArray(i, array);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}


	@SuppressWarnings("rawtypes")
	private ARRAY getArray(Connection con, String OracleObj, String Oraclelist, List<PresApplyRecords> listData) throws Exception {
		ARRAY array = null;
		ArrayDescriptor desc = ArrayDescriptor.createDescriptor(Oraclelist, con);
		STRUCT[] structs = new STRUCT[listData.size()];
		if (listData != null && listData.size() > 0) {
			StructDescriptor structdesc = new StructDescriptor(OracleObj, con);
			for (int i = 0; i < listData.size(); i++) {
				PresApplyRecords presApplyRecords = listData.get(i);
				Object[] result = {presApplyRecords.getPresApplyNo(),presApplyRecords.getItemCode()};
				structs[i] = new STRUCT(structdesc, con, result);
			}
			array = new ARRAY(desc, con, structs);
		} else {
			array = new ARRAY(desc, con, structs);
		}
		return array;
	}

	@Override
	public Object getNullableResult(ResultSet resultSet, String s) throws SQLException {
		return null;
	}

	@Override
	public Object getNullableResult(ResultSet resultSet, int i) throws SQLException {
		return null;
	}


	/**
	 * 获取值部分 接收返回值
	 * @param callableStatement
	 * @param i
	 * @return
	 * @throws SQLException
	 */
	@Override
	public Object getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
		List<PresApplyRecords> objList = new ArrayList<>();
		try (Connection conn = callableStatement.getConnection()) {

			if (null != conn) {
				ResultSet rs = callableStatement.getArray(i).getResultSet();
				while (rs.next()) {
					Datum[] d = ((STRUCT) rs.getObject(2)).getOracleAttributes();
					PresApplyRecords event = new PresApplyRecords();
					event.setPresApplyNo(d[0].intValue());
					event.setItemCode(d[1].stringValue());

					objList.add(event);
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return objList;
	}
}

```
### mybatis接收游标
两种方式
1. 直接用hashmap接收
2. 使用对象映射

```
<!--映射接收游标 转对象 return list-->
<resultMap id="type_pres_apply_records" type="com.zoe.optimus.service.prescribe.entity.prescribe.PresApplyRecords">
        <result property="presApplyNo" column="pres_apply_no"/>
        <result property="presNo" column="pres_no"/>
        <result property="presSubNo" column="pres_sub_no"/>
        <result property="execTime" column="exec_time"/>
        <result property="freqCode" column="freq_name"/>
        <result property="freqName" column="untuck_freq_flag"/>
        <result property="times" column="times"/>
        <result property="presApplyStatus" column="pres_apply_status"/>
    </resultMap>

 <--hashmap接收游标 return map-->
<resultMap type="java.util.HashMap" id="type_pres_apply_records"/>
```
### 调用存储过程写法

```
 <select id="execApplysForBackGroundPackage1" parameterType="map" statementType="CALLABLE">
         call zoepackage.ZOE_APPLYS.test(
         #{map.param_data, jdbcType=ARRAY, mode=IN, typeHandler=com.zoe.optimus.service.config.PresApplyRecordsArrayTypeHandler},
         #{map.operator, jdbcType=VARCHAR, mode=IN},
         #{map.replenisFlag, jdbcType=BOOLEAN, mode=IN},
         #{map.valtest, jdbcType=VARCHAR, mode=OUT},
         #{map.paramLis, jdbcType=CURSOR, mode=OUT, resultMap=type_pres_apply_records}
          )
    </select>
```
需要注意 自定义数组需要定义typeHandler
目前mybatis这种方式 有一个问题 我还没有解决 :
传入boolean类型无法识别 所以改为int类型操作