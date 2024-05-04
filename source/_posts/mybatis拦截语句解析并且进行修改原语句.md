---
title: mybatis拦截语句解析并且进行修改原语句
date: 2024-05-04 15:45:27
tags: [mybatis]
---
## 引入依赖

    <!--语法解析包-->
       <dependency>
                <groupId>com.github.jsqlparser</groupId>
                <artifactId>jsqlparser</artifactId>
                <version>4.6</version>
            </dependency>


            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.3.0</version>
            </dependency>

## 编写拦截器
<!--more-->
```
package com.linjingc.MybatisChange.config;

import cn.hutool.core.util.StrUtil;
import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlCommandType;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.reflection.DefaultReflectorFactory;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.SystemMetaObject;
import org.apache.ibatis.session.Configuration;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.Field;
import java.sql.Connection;
import java.util.Properties;

@Intercepts({@Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class})})
//分页插件后执行
//@AutoConfigureBefore({PageHelperAutoConfiguration.class})
public class MybatisChangeSqlInterceptor implements Interceptor {
    public MybatisChangeSqlInterceptor(String dbType) {
        this.dbType = dbType;
    }

    //数据库类型
    private static String dbType;
    private static final Logger logger = LoggerFactory.getLogger(MybatisChangeSqlInterceptor.class);

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("====生成构建语句拦截intercept======");

        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
        //通过MetaObject优雅访问对象的属性，这里是访问statementHandler的属性;：MetaObject是Mybatis提供的一个用于方便、
        //优雅访问对象属性的对象，通过它可以简化代码、不需要try/catch各种reflect异常，同时它支持对JavaBean、Collection、Map三种类型对象的操作。
        MetaObject metaObject = MetaObject.forObject(statementHandler, SystemMetaObject.DEFAULT_OBJECT_FACTORY, SystemMetaObject.DEFAULT_OBJECT_WRAPPER_FACTORY, new DefaultReflectorFactory());
        MappedStatement mappedStatement = (MappedStatement) metaObject.getValue("delegate.mappedStatement");
        // id为执行的mapper方法的全路径名，如com.mapper.UserMapper
        String id = mappedStatement.getId();
        // sql语句类型 select、delete、insert、update
        String sqlCommandType = mappedStatement.getSqlCommandType().toString();
        logger.info("执行的mapper方法的全路径名: {}", id);
        logger.info("sql语句类型: {}", sqlCommandType);
        // 仅拦截 select 查询
        if (!sqlCommandType.equals(SqlCommandType.SELECT.toString())) {
            logger.info("仅拦截 select 查询");
            return invocation.proceed();
        }
        //数据库连接信息
        Configuration configuration = mappedStatement.getConfiguration();
        HikariDataSource dataSource = (HikariDataSource) configuration.getEnvironment().getDataSource();
        logger.info("数据库地址:{}", dataSource.getJdbcUrl());

        BoundSql boundSql = statementHandler.getBoundSql();
        //获取到原始sql语句
        String sql = boundSql.getSql();
        sql = StrUtil.removeAll(sql, "\n");
        sql = sql.replaceAll("\\s+", " ");
        logger.info("原始SQL: {}", sql);

        // 修改sql内容
        String modifiedSql = modifySql(sql);
        //通过反射修改sql语句
        Field field = boundSql.getClass().getDeclaredField("sql");
        field.setAccessible(true);
        field.set(boundSql, modifiedSql);


        //执行结果
        return invocation.proceed();

    }


    /**
     * 实现此方法以修改SQL
     *
     * @return
     */
    private String modifySql(String originalSql) {
        // 这里可以根据需要修改SQL
        //根据dbType进行修改sql
        SqlCaseUtils.setDbType(dbType);
        String sql =SqlCaseUtils.changeSql(originalSql);
        logger.info("修改后语句:{}", sql);
        return sql;
    }


    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }


}


```

## 注册拦截器

```
package com.linjingc.MybatisChange.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import java.util.List;

@Configuration
public class MybatisConfig {
    @Autowired
    private List<SqlSessionFactory> sqlSessionFactoryList;

    /**
     * mybatis 拦截器注册
     */
    @PostConstruct
    public void addSqlInterceptor() {
        MybatisChangeSqlInterceptor mybatisChangeSqlInterceptor = new MybatisChangeSqlInterceptor("dm");
        for (SqlSessionFactory sqlSessionFactory : sqlSessionFactoryList) {
            sqlSessionFactory.getConfiguration().addInterceptor(mybatisChangeSqlInterceptor);
        }
    }
}

```

## 语法解析工具类

```
package com.linjingc.MybatisChange.config;

import net.sf.jsqlparser.JSQLParserException;
import net.sf.jsqlparser.expression.Expression;
import net.sf.jsqlparser.expression.ExpressionVisitorAdapter;
import net.sf.jsqlparser.expression.operators.arithmetic.Concat;
import net.sf.jsqlparser.expression.operators.conditional.OrExpression;
import net.sf.jsqlparser.expression.operators.relational.Between;
import net.sf.jsqlparser.expression.operators.relational.EqualsTo;
import net.sf.jsqlparser.expression.operators.relational.IsNullExpression;
import net.sf.jsqlparser.expression.operators.relational.LikeExpression;
import net.sf.jsqlparser.parser.CCJSqlParserUtil;
import net.sf.jsqlparser.schema.Column;
import net.sf.jsqlparser.statement.Statement;
import net.sf.jsqlparser.statement.select.*;
import net.sf.jsqlparser.util.TablesNamesFinder;

import java.util.List;

/**
 * sql语句转换处理工具类
 */
public class SqlCaseUtils {
    public final static String ORACLE = "oracle";
    public final static String DM = "dm";
    public final static String KING_BASE = "kingbase";
    public final static String SQL_SERVER = "sqlserver";
    public final static String MYSQL = "mysql";


    public static void setDbType(String dbType) {
        SqlCaseUtils.dbType = dbType;
    }

    /**
     * 当前数据库类型
     */
    private static String dbType;

    /**
     * 判断字段为空时转换值
     *
     * @param column_name 字段名
     * @param patten      转换值
     * @return
     */
    public static String isNull(String column_name, String patten) {
        String reuslt = "";
        if (column_name != null && !column_name.equals("")) {
            switch (dbType) {
                case ORACLE:
                    reuslt = "nvl(" + column_name + "," + patten + ")";
                    break;
                case DM:
                    reuslt = "nvl(" + column_name + "," + patten + ")";
                    break;
                case KING_BASE:
                    reuslt = "isnull(" + column_name + "," + patten + ")";
                    break;
                case SQL_SERVER:
                    reuslt = "isnull(" + column_name + "," + patten + ")";
                    break;
                case MYSQL:
                    reuslt = "ifnull(" + column_name + "," + patten + ")";
                    break;
            }
        } else {
            return "column_name不能为空！";
        }
        return reuslt;
    }

    /**
     * 获取当前日期
     *
     * @return
     */
    public static String sysDate() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "sysdate";
                break;
            case DM:
                reuslt = "now()";
                break;
            case KING_BASE:
                reuslt = "now()";
                break;
            case SQL_SERVER:
                reuslt = "getdate()";
                break;
            case MYSQL:
                reuslt = "sysdate()";
                break;
        }
        return reuslt;
    }

    /**
     * 格式转换[时间类型转字符串]
     *
     * @return
     */
    public static String dateToStr(String column_name, int ws) {
        String reuslt = "";
        String dateStr = "'" + getDateStr(ws) + "'";
        switch (dbType) {
            case ORACLE:
                reuslt = "to_char(" + column_name + "," + dateStr + ")";
                break;
            case DM:
                reuslt = "to_char(" + column_name + "," + dateStr + ")";
                break;
            case KING_BASE:
                reuslt = "to_char(" + column_name + "," + dateStr + ")";
                break;
            case SQL_SERVER:
                if (ws == -1) {
                    reuslt = "convert(varchar," + column_name + " ,121)";
                } else {
                    reuslt = "substring(convert(varchar," + column_name + " ,121),0," + (ws + 1) + ")";
                }

                break;
            case MYSQL:
                reuslt = " SUBSTR((cast(" + column_name + " as char)) ,1," + ws + ") ";
                break;
        }
        return reuslt;
    }

    /**
     * 格式转换[时间类型转字符串]带点的
     *
     * @return
     */
    public static String dateToStrD(String column_name, int ws) {
        String reuslt = "";
        String dateStr = "'" + getDateStrD(ws) + "'";
        switch (dbType) {
            case ORACLE:
                reuslt = "to_char(" + column_name + "," + dateStr + ")";
                break;
            case DM:
                reuslt = "to_char(" + column_name + "," + dateStr + ")";
                break;
            case KING_BASE:
                reuslt = "to_char(" + column_name + "," + dateStr + ")";
                break;
            case SQL_SERVER:
                reuslt = "substring(convert(varchar(255)," + column_name + ",121),0," + (ws + 1) + ")";
                break;
            case MYSQL:
                reuslt = " SUBSTR((cast(" + column_name + " as char)) ,1," + ws + ") ";
                break;
        }
        return reuslt;
    }


    public static String getDateStr(int ws) {
        String result = "";
        String str = "";
        switch (dbType) {
            case ORACLE:
                str = "yyyy-mm-dd hh24:mi:ss";
                if (ws >= 13) {
                    ws += 2;
                }
                result = str.substring(0, ws);
                break;
            case DM:
                str = "yyyy-mm-dd hh24:mi:ss";
                if (ws >= 13) {
                    ws += 2;
                }
                result = str.substring(0, ws);
                break;
            case KING_BASE:
                str = "yyyy-mm-dd hh24:mi:ss";
                if (ws >= 13) {
                    ws += 2;
                }
                result = str.substring(0, ws);
                break;
            case SQL_SERVER:
                str = "yyyy-MM-dd HH:mm:ss";
                result = ws != -1 ? str.substring(0, ws) : str;
                break;
            case MYSQL:
                str = "%Y-%M-%D %h:%i:%s";
                result = str.substring(0, ws - 2);
                break;
        }
        return result;

    }

    public static String getDateStrD(int ws) {
        String result = "";
        String str = "";
        switch (dbType) {
            case ORACLE:
                str = "yyyy.mm.dd hh24:mi:ss";
                if (ws >= 13) {
                    ws += 2;
                }
                result = str.substring(0, ws);
                break;
            case DM:
                str = "yyyy.mm.dd hh24:mi:ss";
                if (ws >= 13) {
                    ws += 2;
                }
                result = str.substring(0, ws);
                break;
            case KING_BASE:
                str = "yyyy.mm.dd hh24:mi:ss";
                if (ws >= 13) {
                    ws += 2;
                }
                result = str.substring(0, ws);
                break;
            case SQL_SERVER:
                str = "yyyy.MM.dd HH:mm:ss";
                result = str.substring(0, ws);
                break;
            case MYSQL:
                str = "%Y.%M.%D %h:%i:%s";
                result = str.substring(0, ws - 2);
                break;
        }
        return result;

    }


    /**
     * 格式转换[字符串转时间类型]
     *
     * @return
     */
    public static String strToDate(String column_name, int ws) {
        String reuslt = "";
        String dateStr = "'" + getDateStr(ws) + "'";
        switch (dbType) {
            case ORACLE:
                reuslt = "TO_DATE(" + column_name + "," + dateStr + ")";
                break;
            case DM:
                reuslt = "TO_DATE(" + column_name + "," + dateStr + ")";
                break;
            case KING_BASE:
                reuslt = "TO_DATE(" + column_name + "," + dateStr + ")";
                break;
            case SQL_SERVER:
                reuslt = "convert(datetime," + column_name + ",121)";
                break;
            case MYSQL:
                reuslt = "cast(" + column_name + " as datetime)";
                break;
        }
        return reuslt;
    }


    /**
     * 格式转换[字符串转时间类型]
     *
     * @return
     */
    public static String strToNumber(String column_name, String patten) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "CAST(" + column_name + " as NUMBER" + patten + ")";
                break;
            case DM:
                reuslt = "CAST(" + column_name + " as NUMBER" + patten + ")";
                break;
            case KING_BASE:
                reuslt = "CAST(" + column_name + " as int)";
                break;
            case SQL_SERVER:
                reuslt = "CAST(" + column_name + " as int)";
                break;
            case MYSQL:
                reuslt = "CAST(" + column_name + " as NUMBER" + patten + ")";
                break;
        }
        return reuslt;
    }

    /**
     * 格式转换[字符串转时间类型]
     *
     * @return
     */
    public static String strToFloat(String column_name, String patten) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "CAST(" + column_name + " as FLOAT" + patten + ")";
                break;
            case DM:
                reuslt = "CAST(" + column_name + " as FLOAT" + patten + ")";
                break;
            case KING_BASE:
                reuslt = "CAST(" + column_name + " as FLOAT)";
                break;
            case SQL_SERVER:
                reuslt = "CAST(" + column_name + " as FLOAT)";
                break;
            case MYSQL:
                reuslt = "CAST(" + column_name + " as FLOAT" + patten + ")";
                break;
        }
        return reuslt;
    }

    /**
     * 格式转换[其他类型转字符串]
     *
     * @return
     */
    public static String castToStr(String column_name) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "to_char(" + column_name + ")";
                break;
            case DM:
                reuslt = "to_char(" + column_name + ")";
                break;
            case KING_BASE:
                reuslt = "to_char(" + column_name + ")";
                break;
            case SQL_SERVER:
                reuslt = "convert(varchar(255)," + column_name + ")";
                break;
            case MYSQL:
                reuslt = "convert(" + column_name + ",char)";
                break;
        }
        return reuslt;
    }

    /**
     * 分页
     *
     * @param table_name   表名
     * @param column_names 字段名
     * @param where        where条件
     * @param order        排序
     * @param pageNo       页数
     * @param pageCount    每页条数
     * @return
     */
    public static String makePaging(String table_name, String column_names, String where, String order, int pageNo, int pageCount) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "SELECT " + column_names + " FROM " + table_name + " T1, (SELECT RID FROM (SELECT ROWNUM RN, T.RID FROM (SELECT ROWID RID FROM " + table_name + " " + where + " " + order + " ) T WHERE  ROWNUM <= " + (pageNo) * pageCount + ") WHERE RN > " + (pageNo - 1) * pageCount + ") T2 WHERE T1.ROWID = T2.RID " + order + "";
                break;
            case DM:
                reuslt = "SELECT " + column_names + " FROM " + table_name + " T1, (SELECT RID FROM (SELECT ROWNUM RN, T.RID FROM (SELECT ROWID RID FROM " + table_name + " " + where + "  " + order + " ) T WHERE  ROWNUM <= " + (pageNo) * pageCount + ") WHERE RN > " + (pageNo - 1) * pageCount + ") T2 WHERE T1.ROWID = T2.RID   " + order + "";
                break;
            case KING_BASE:
                reuslt = "select " + column_names + " from " + table_name + "  " + where + " " + order + " limit " + (pageNo - 1) * pageCount + "," + pageCount;
                break;
            case SQL_SERVER:
                reuslt = "select " + column_names + " from " + table_name + " " + table_name + " " + where + "   " + order + " offset " + (pageNo - 1) * pageCount + " rows fetch next " + pageCount + " rows only";
                break;
            case MYSQL:
                reuslt = "select " + column_names + " from " + table_name + "  " + where + " " + order + " limit " + (pageNo - 1) * pageCount + "," + (pageNo) * pageCount;
                break;
        }
        return reuslt;
    }


    /**
     * 获取表中的第一条或最新一条数据
     *
     * @param table_name   表名
     * @param column_names 字段名
     * @param where        where条件，可以为空
     * @param order        排序条件，确定数据的"第一条"或"最新一条"
     * @return 查询SQL
     */
    public static String getFirstOrLastRecord(String table_name, String column_names, String where, String order) {
        String result = "";
        System.out.println(dbType);
        switch (dbType) {
            case ORACLE:
                result = "SELECT " + column_names + " FROM (SELECT " + column_names + " FROM " + table_name + (where.isEmpty() ? "" : " WHERE " + where) + " " + order + ") WHERE ROWNUM = 1";
                break;
            case DM:
                // 达梦数据库和Oracle类似，但具体分页方式可能有差异，这里假设和Oracle相同
                result = "SELECT " + column_names + " FROM (SELECT " + column_names + " FROM " + table_name + (where.isEmpty() ? "" : " WHERE " + where) + " " + order + ") WHERE ROWNUM = 1";
                break;
            case KING_BASE:
                result = "SELECT " + column_names + " FROM " + table_name + (where.isEmpty() ? "" : " WHERE " + where) + " " + order + " LIMIT 1";
                break;
            case SQL_SERVER:
                result = "SELECT TOP 1 " + column_names + " FROM " + table_name + (where.isEmpty() ? "" : " WHERE " + where) + " " + order;
                break;
            case MYSQL:
                result = "SELECT " + column_names + " FROM " + table_name + (where.isEmpty() ? "" : " WHERE " + where) + " " + order + " LIMIT 1";
                break;
        }
        return result;
    }


    /**
     * 分页
     *
     * @param table_name   表名
     * @param column_names 字段名
     * @param where        where条件
     * @param order        排序
     * @param pageNo       页数
     * @param pageCount    每页条数
     * @return
     */
    public static String makePagingForSQLTable(String table_name, String column_names, String where, String order, int pageNo, int pageCount) {
        String reuslt = "";
        System.out.println(dbType);
        switch (dbType) {
            case ORACLE:
                reuslt = "SELECT " + column_names + " FROM " + table_name + " T1, (SELECT RID FROM (SELECT ROWNUM RN, T.RID FROM (SELECT TOM_ROWID RID FROM " + table_name + " " + where + " " + order + " ) T WHERE  ROWNUM <= " + (pageNo) * pageCount + ") WHERE RN > " + (pageNo - 1) * pageCount + ") T2 WHERE T1.ROWID = T2.RID " + order + "";
                break;
            case DM:
                reuslt = "SELECT " + column_names + " FROM " + table_name + " T1, (SELECT RID FROM (SELECT ROWNUM RN, T.RID FROM (SELECT TOM_ROWID RID FROM " + table_name + " " + where + "  " + order + " ) T WHERE  ROWNUM <= " + (pageNo) * pageCount + ") WHERE RN > " + (pageNo - 1) * pageCount + ") T2 WHERE T1.TOM_ROWID = T2.RID   " + order + "";
                break;
            case KING_BASE:
                reuslt = "select " + column_names + " from " + table_name + "  " + where + " " + order + " limit " + (pageNo - 1) * pageCount + "," + pageCount;
                break;
            case SQL_SERVER:
                reuslt = "select " + column_names + " from " + table_name + " T1   " + order + " offset " + (pageNo - 1) * pageCount + " rows fetch next " + pageCount + " rows only";
                break;
            case MYSQL:
                reuslt = "select " + column_names + " from " + table_name + "  " + where + " " + order + " limit " + (pageNo - 1) * pageCount + "," + (pageNo) * pageCount;
                break;
        }
        return reuslt;
    }

    public static String makePagingAfterOrder(String table_name, String column_names, String where, String order, int pageNo, int pageCount) {
        String reuslt = "";
        System.out.println(dbType);
        switch (dbType) {
            case ORACLE:
                reuslt = "SELECT " + column_names + " FROM " + table_name + " T1, (SELECT RID FROM (SELECT ROWNUM RN, T.RID FROM (SELECT ROWID RID FROM " + table_name + " " + where + "  ) T WHERE  ROWNUM <= " + (pageNo) * pageCount + ") WHERE RN > " + (pageNo - 1) * pageCount + ") T2 WHERE T1.ROWID = T2.RID " + order + " ";
                break;
            case DM:
                reuslt = "SELECT " + column_names + " FROM " + table_name + " T1, (SELECT RID FROM (SELECT ROWNUM RN, T.RID FROM (SELECT ROWID RID FROM " + table_name + " " + where + "   ) T WHERE  ROWNUM <= " + (pageNo) * pageCount + ") WHERE RN > " + (pageNo - 1) * pageCount + ") T2 WHERE T1.ROWID = T2.RID  " + order + " ";
                break;
            case KING_BASE:
                reuslt = "select " + column_names + " from " + table_name + " T1  " + where + " " + order + " limit " + (pageNo - 1) * pageCount + "," + pageCount;
                break;
            case SQL_SERVER:
                reuslt = "select " + column_names + " from " + table_name + " T1   " + order + " offset " + (pageNo - 1) * pageCount + " rows fetch next " + pageCount + " rows only";
                break;
            case MYSQL:
                reuslt = "select " + column_names + " from " + table_name + " T1  " + where + " " + order + " limit " + (pageNo - 1) * pageCount + "," + (pageNo) * pageCount;
                break;
        }
        return reuslt;
    }


    /**
     * 截取字符串
     *
     * @return
     */
    public static String subString(String str, int start, int end) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "substr(" + str + "," + start + "," + end + ")";
                break;
            case DM:
                reuslt = "substr(" + str + "," + start + "," + end + ")";
                break;
            case KING_BASE:
                reuslt = "substring(" + str + "," + start + "," + end + ")";
                break;
            case SQL_SERVER:
                reuslt = "substring(" + str + "," + start + "," + end + ")";
                break;
            case MYSQL:
                reuslt = "substr(" + str + "," + start + "," + end + ")";
                break;
        }
        return reuslt;
    }


    /**
     * 拼接字符串
     *
     * @return
     */
    public static String translate(String sourceStr, String targetStr, String replaceStr) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "translate(" + sourceStr + "," + targetStr + "," + replaceStr + ")";
                break;
            case DM:
                reuslt = "translate(" + sourceStr + "," + targetStr + "," + replaceStr + ")";
                break;
            case KING_BASE:
                reuslt = "translate(" + sourceStr + "," + targetStr + "," + replaceStr + ")";
                break;
            case SQL_SERVER:
                reuslt = "TRANSLATE(" + sourceStr + "," + targetStr + "," + replaceStr + ")";
                break;
            case MYSQL:
                reuslt = "REGEXP_REPLACE(" + sourceStr + "," + targetStr + "," + replaceStr + ")";
                break;
        }
        return reuslt;
    }


    /**
     * 拼接字符串
     *
     * @return
     */
    public static String appendStr(String sql) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "WMSYS.WM_CONCAT(" + sql + ")";
                break;
            case DM:
                reuslt = "WMSYS.WM_CONCAT(" + sql + ")";
                break;
            case KING_BASE:
                reuslt = "WM_CONCAT(" + sql + ")";
                break;
            case SQL_SERVER:
                reuslt = "";
                break;
            case MYSQL:
                reuslt = "WMSYS.WM_CONCAT(" + sql + ")";
                break;
        }
        return reuslt;
    }

    public static String getUUID() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "rawtohex(sys_guid())";
                break;
            case DM:
                reuslt = "rawtohex(sys_guid())";
                break;
            case KING_BASE:
                reuslt = "sys_guid()";
                break;
            case SQL_SERVER:
                reuslt = "CAST(NEWID() AS VARCHAR(36))";
                break;
            case MYSQL:
                reuslt = "UUID()";
                break;
        }
        return reuslt;
    }

    public static String getVarchar() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "varchar2";
                break;
            case DM:
                reuslt = "varchar2";
                break;
            case KING_BASE:
                reuslt = "varchar2";
                break;
            case SQL_SERVER:
                reuslt = "varchar";
                break;
            case MYSQL:
                reuslt = "varchar";
                break;
        }
        return reuslt;
    }

    public static String getInt() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "int";
                break;
            case DM:
                reuslt = "int";
                break;
            case KING_BASE:
                reuslt = "int";
                break;
            case SQL_SERVER:
                reuslt = "int";
                break;
            case MYSQL:
                reuslt = "int";
                break;
        }
        return reuslt;
    }

    public static String getLong() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "long";
                break;
            case DM:
                reuslt = "bigint";
                break;
            case KING_BASE:
                reuslt = "bigint";
                break;
            case SQL_SERVER:
                reuslt = "bigint";
                break;
            case MYSQL:
                reuslt = "bigint";
                break;
        }
        return reuslt;
    }


    public static String getFloat() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "float";
                break;
            case DM:
                reuslt = "float";
                break;
            case KING_BASE:
                reuslt = "float";
                break;
            case SQL_SERVER:
                reuslt = "float";
                break;
            case MYSQL:
                reuslt = "float";
                break;
        }
        return reuslt;
    }

    public static String getMoney() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "number(10,4)";
                break;
            case DM:
                reuslt = "number(10,4)";
                break;
            case KING_BASE:
                reuslt = "number(10,4)";
                break;
            case SQL_SERVER:
                reuslt = "numeric(10,4)";
                break;
            case MYSQL:
                reuslt = "numeric(10,4)";
                break;
        }
        return reuslt;
    }

    public static String getNumeric(int length, int accuracy) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "number(" + length + "," + accuracy + ")";
                break;
            case DM:
                reuslt = "number(" + length + "," + accuracy + ")";
                break;
            case KING_BASE:
                reuslt = "number(" + length + "," + accuracy + ")";
                break;
            case SQL_SERVER:
                reuslt = "numeric(" + length + "," + accuracy + ")";
                break;
            case MYSQL:
                reuslt = "numeric(" + length + "," + accuracy + ")";
                break;
        }
        return reuslt;
    }

    public static String getDateTime() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "timestamp";
                break;
            case DM:
                reuslt = "datetime";
                break;
            case KING_BASE:
                reuslt = "timestamp";
                break;
            case SQL_SERVER:
                reuslt = "datetime";
                break;
            case MYSQL:
                reuslt = "datetime";
                break;
        }
        return reuslt;
    }

    public static String listAgg(String field) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "listagg ( " + field + ", ',' ) within GROUP ( ORDER BY " + field + " )";
                break;
            case DM:
                reuslt = "listagg ( " + field + ", ',' ) within GROUP ( ORDER BY " + field + " )";
                break;
            case KING_BASE:
                reuslt = "string_agg ( " + field + ",',' order by " + field + ")";
                break;
            case SQL_SERVER:
                reuslt = "string_agg ( " + field + ", ',' ) within GROUP ( ORDER BY " + field + " )";
                break;
            case MYSQL:
                reuslt = "group_concat( " + field + ",'' order by " + field + ")";
                break;
        }
        return reuslt;
    }

    /**
     * 为日期增加指定月份数
     *
     * @param column_name   要操作的列名
     * @param months_to_add 要增加的月份数，可以是负数
     * @return 增加月份后的日期的SQL表达式
     */
    public static String addMonths(String column_name, int months_to_add) {
        String result = "";
        switch (dbType) {
            case ORACLE:
                // Oracle数据库直接支持ADD_MONTHS函数
                result = "ADD_MONTHS(" + column_name + ", " + months_to_add + ")";
                break;
            case DM:
                // 达梦数据库，假设和Oracle一致
                result = "ADD_MONTHS(" + column_name + ", " + months_to_add + ")";
                break;
            case KING_BASE:
                // 人大金仓数据库，假设和Oracle一致
                result = "ADD_MONTHS(" + column_name + ", " + months_to_add + ")";
                break;
            case SQL_SERVER:
                // SQL Server使用DATEADD函数
                result = "DATEADD(MONTH, " + months_to_add + ", " + column_name + ")";
                break;
            case MYSQL:
                // MySQL使用DATE_ADD函数
                result = "DATE_ADD(" + column_name + ", INTERVAL " + months_to_add + " MONTH)";
                break;
            case "postgresql":
                // PostgreSQL使用间隔表达式
                result = column_name + " + INTERVAL '" + months_to_add + " months'";
                break;
        }
        return result;
    }


    /**
     * 根据不同数据库生成对应的日期操作SQL
     *
     * @param days 天数，用于计算从当前日期向前推的天数
     * @return 对应数据库的日期操作SQL
     */
    public static String generateDateOperationSQL(int days) {
        String sql = "";
        switch (dbType) {
            case ORACLE:
                // 原始Oracle SQL已经给出，这里不再重复
                sql = "TRUNC(sysdate() - NUMTODSINTERVAL(" + days + ", 'DAY')) + LEVEL";
                break;
            case MYSQL:
            case "mariadb":
                // MySQL和MariaDB使用DATE_SUB和DATE函数
                sql = "SELECT DATE(DATE_SUB(NOW(), INTERVAL " + days + " DAY))";
                // MySQL和MariaDB不支持LEVEL，递归逻辑需要另外构建
                break;
            case "postgresql":
                // PostgreSQL使用DATE_TRUNC和INTERVAL
                sql = "SELECT DATE_TRUNC('day', CURRENT_DATE - INTERVAL '" + days + " day')::date";
                // PostgreSQL可以使用generate_series来生成日期序列
                break;
            case SQL_SERVER:
                // SQL Server使用DATEADD和CAST
                sql = "SELECT CAST(DATEADD(DAY, -" + days + ", GETDATE()) AS DATE)";
                // SQL Server同样可以使用递归CTE来生成日期序列
                break;
        }
        return sql;
    }

    /**
     * 获取字符串长度的SQL表达式
     *
     * @param column_name 要操作的列名或字符串
     * @return 获取长度后的SQL表达式
     */
    public static String getLengthSQL(String column_name) {
        String result = "";
        switch (dbType) {
            case ORACLE:
            case DM:
            case "postgresql":
                // Oracle, 达梦(DM) 和 PostgreSQL 使用 LENGTH 函数
                result = "LENGTH(" + column_name + ")";
                break;
            case SQL_SERVER:
                // SQL Server 使用 LEN 函数
                result = "LEN(" + column_name + ")";
                break;
            case MYSQL:
            case "mariadb":
                // MySQL 和 MariaDB 使用 CHAR_LENGTH 函数（或 LENGTH，注意在MySQL中，LENGTH()返回的是字节长度，对于多字节字符集可能与字符数不同）
                result = "CHAR_LENGTH(" + column_name + ")";
                break;
            case KING_BASE:
                // 人大金仓数据库，假设使用 LENGTH 函数，具体可能需要验证
                result = "LENGTH(" + column_name + ")";
                break;
        }
        return result;
    }

    /**
     * 生成对应数据库的LPAD函数SQL表达式
     *
     * @param column_name 要操作的列名或字符串
     * @param length      填充后的长度
     * @param pad_string  填充使用的字符串
     * @return LPAD函数的SQL表达式
     */
    public static String getLpadSQL(String column_name, int length, String pad_string) {
        String result = "";
        switch (dbType) {
            case ORACLE:
            case DM:
            case "postgresql":
            case MYSQL:
            case "mariadb":
            case KING_BASE:
                // 这些数据库直接支持LPAD函数
                result = "LPAD(" + column_name + ", " + length + ", '" + pad_string + "')";
                break;
            case SQL_SERVER:
                // SQL Server不直接支持LPAD，可以使用REPLICATE和LEFT函数组合来模拟LPAD行为
                result = "RIGHT(REPLICATE('" + pad_string + "', " + length + ") + " + column_name + ", " + length + ")";
                break;
        }
        return result;
    }


    /**
     * 拼接字符串
     *
     * @return
     */
    public static String append() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "||";
                break;
            case DM:
                reuslt = "||";
                break;
            case KING_BASE:
                reuslt = "+";
                break;
            case SQL_SERVER:
                reuslt = "+";
                break;
            case MYSQL:
                reuslt = "+";
                break;
        }
        return reuslt;
    }

    public static String getModify() {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "modify";
                break;
            case DM:
                reuslt = "modify";
                break;
            case KING_BASE:
                reuslt = "modify";
                break;
            case SQL_SERVER:
                reuslt = "ALTER COLUMN";
                break;
            case MYSQL:
                reuslt = "modify";
                break;
        }
        return reuslt;
    }


    /**
     * 根据不同数据库生成对应的日期操作SQL 获取年份
     *
     * @param columnName
     * @return
     */
    public static String getYear(String columnName) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = " EXTRACT(YEAR FROM " + columnName + ") ";
                break;
            case DM:
                reuslt = " EXTRACT(YEAR FROM " + columnName + ") ";
                break;
            case KING_BASE:
                reuslt = " EXTRACT(YEAR FROM " + columnName + ") ";
                break;
            case SQL_SERVER:
                reuslt = " YEAR(" + columnName + ") ";
                break;
            case MYSQL:
                reuslt = " EXTRACT(YEAR FROM " + columnName + ") ";
                break;
        }
        return reuslt;
    }

    /**
     * 根据不同数据库生成对应的查询字符串中出现几次子字符串
     *
     * @param column 字段名
     * @param text   需要查询的文本
     * @param
     * @return
     */
    public static String getExistCount(String column, String text) {
        //默认大于0次
        return getExistCount(column, text, 0);
    }

    /**
     * 根据不同数据库生成对应的查询字符串中出现几次子字符串
     *
     * @param column    字段名
     * @param text      需要查询的文本
     * @param countSize 出现的次数大于几次
     * @return
     */
    public static String getExistCount(String column, String text, int countSize) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "INSTR(" + column + ", '" + text + "') > " + countSize;
                break;
            case DM:
                reuslt = "INSTR(" + column + ", '" + text + "') > " + countSize;
                break;
            case KING_BASE:
                reuslt = "INSTR(" + column + ", '" + text + "') > " + countSize;
                break;
            case SQL_SERVER:
                reuslt = "CHARINDEX('" + text + "', " + column + ") > " + countSize;
                break;
            case MYSQL:
                reuslt = "INSTR(" + column + ", '" + text + "') > " + countSize;
                break;
        }
        return reuslt;
    }


    /**
     * 建字段转换为数值类型
     *
     * @param column
     * @return
     */
    public static String getToNumber(String column) {
        String reuslt = "";
        switch (dbType) {
            case ORACLE:
                reuslt = "TO_NUMBER(" + column + ")";
                ;
                break;
            case DM:
                reuslt = "TO_NUMBER(" + column + ")";
                ;
                break;
            case KING_BASE:
                reuslt = "TO_NUMBER(" + column + ")";
                ;
                break;
            case SQL_SERVER:
                reuslt = "CAST(" + column + " AS FLOAT)";
                break;
            case MYSQL:
                reuslt = "TO_NUMBER(" + column + ")";
                ;
                break;
        }
        return reuslt;
    }


    /**
     * 修改原有语句改为 对应数据库的执行语句
     *
     * @param originalSql
     * @return
     */
    public static String changeSql(String originalSql) {
        try {
            String sql = originalSql;
            sql="select\n" +
                    "        menu_name,menu_code,menu_order\n" +
                    "from\n" +
                    "        t_menu_mng\n" +
                    "where\n" +
                    "        id in\n" +
                    "        (\n" +
                    "                select\n" +
                    "                        menu_id\n" +
                    "                from\n" +
                    "                        t_menu_auth\n" +
                    "                where\n" +
                    "                        (\n" +
                    "                                (\n" +
                    "                                        auth_type=1\n" +
                    "                                    and auth_id in\n" +
                    "                                        (\n" +
                    "                                                select\n" +
                    "                                                        manager_type\n" +
                    "                                                from\n" +
                    "                                                        tuser2organ\n" +
                    "                                                where\n" +
                    "                                                        usid      ='5003'\n" +
                    "                                                    and organ_type='sbj'\n" +
                    "                                                    and code      = '1001001'\n" +
                    "                                        )\n" +
                    "                                )\n" +
                    "                        )\n" +
                    "                     or\n" +
                    "                        (\n" +
                    "                                auth_type = 0\n" +
                    "                            and auth_id   =5003\n" +
                    "                            and code      ='1001001'\n" +
                    "                        )\n" +
                    "        )\n" +
                    "order by\n" +
                    "\n" +
                    "        menu_code,menu_order" ;
            // 解析SQL语句
            Statement statement = CCJSqlParserUtil.parse(sql);

            // 获取表名的访问器
            TablesNamesFinder tablesNamesFinder = new TablesNamesFinder();
//            for (String table : tablesNamesFinder.getTables(statement)) {
//                System.out.println("Table name: " + table);
//            }

            if (statement instanceof Select) {
                Select selectStatement = (Select) statement;
                PlainSelect selectBody = (PlainSelect) selectStatement.getSelectBody();

                //解析where条件
                Expression where = selectBody.getWhere();
                where.accept(new ExpressionVisitorAdapter() {
                    @Override
                    public void visit(EqualsTo equalsTo) {
                        System.out.println("Column: " + equalsTo.getLeftExpression() + "     Value: " + equalsTo.getRightExpression());
                    }
                });


                //解析select查询的字段
                List<SelectItem> selectItems = selectBody.getSelectItems();
                selectItems.forEach(selectItem -> selectItem.accept(new SelectItemVisitorAdapter() {
                    @Override
                    public void visit(AllColumns columns) {
                        super.visit(columns);
                        System.out.println("查询的字段Column: " + columns.toString());

                    }

                    @Override
                    public void visit(AllTableColumns columns) {
                        super.visit(columns);
                        System.out.println("查询的字段Column: " + columns.toString());

                    }

                    @Override
                    public void visit(SelectExpressionItem item) {
                        super.visit(item);
                    }

//                    @Override
//                    public void visit(SelectItem selectItem) {
//                        if (selectItem.getExpression() instanceof Column) {
//                            Column column = (Column) selectItem.getExpression();
//                            System.out.println("查询的字段Column: " + column.getFullyQualifiedName());
//                        }
//                        //表示这个查询语句使用查询*号来处理的
//                        if (selectItem.getExpression() instanceof AllColumns) {
//                            AllColumns column = (AllColumns) selectItem.getExpression();
//                            System.out.println("查询的字段Column: " + column.toString());
//                        }
//
//                    }
                }));


            }

        } catch (JSQLParserException e) {
            throw new RuntimeException(e);
        }


        return originalSql;
    }
}

```

