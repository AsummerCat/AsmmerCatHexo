---
title: Go语言连接Mysql数据库
date: 2022-10-19 14:13:03
tags: [go]
---
# Go语言连接Mysql数据库

Go语言的`database/sql`包提供了保证sql的泛用接口,不提供具体实现

常用第三方驱动包 ->跟java类似

## 下载依赖
```
go get -u github.com/go-sql-driver/mysql
```
## 如果无法下载
```
这是因为无法对外网进行访问，可以通过输入一下命令进入代理网站进行访问

go env -w GOPROXY=https://goproxy.cn,direct
```
## 使用Mysql驱动
```
func Open(driverName, DataSourceName string)(*DB, error)

```
# 基础使用
<!--more-->
## 1.参数:SetMaxOpenConns
```
func (db *DB) SetMaxOpenConns(n int)
```
SetMaxOpenConns设置与数据库建立连接的最大数目。 如果n大于0且小于最大闲置连接数，会将最大闲置连接数减小到匹配最大开启连接数的限制。 如果n<=0，不会限制最大开启连接数，默认为0（无限制）。

## 2.参数:SetMaxIdleConns
```
SetMaxIdleConns设置连接池中的最大闲置连接数。 如果n大于最大开启连接数，则新的最大闲置连接数会减小到匹配最大开启连接数的限制。 如果n<=0，不会保留闲置连接。
```

## 3.单行查询
单行查询`db.QueryRow()`执行一次查询，并期望返回最多一行结果（即Row）。QueryRow总是返回非nil的值，直到返回值的Scan方法被调用时，才会返回被延迟的错误。（如：未找到结果）

## 4.多行查询
多行查询`db.Query()`执行一次查询，返回多行结果（即Rows），一般用于执行select命令。参数args表示query中的占位参数。

## 5.插入数据
```
插入、更新和删除操作都使用Exec方法。
func (db *DB) Exec(query string, args ...interface{}) (Result, error)
```
## 案例
```
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

var db *sql.DB

/************数据库的基本使用*********/

/*
初始化数据库
*/
func initBb() (db *sql.DB) {
	dsn := "root:password@tcp(47.37.426.110:3306)/cy?charset=utf8"
	//打开数据库连接
	db, err := sql.Open("mysql", dsn)
	//设置链接数
	db.SetMaxOpenConns(10)
	//设置连接池最大限制链接数
	db.SetMaxIdleConns(100)
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		fmt.Printf("尝试与数据库建立连接失败", err)
	}
	if err != nil {
		fmt.Printf("连接数据库失败", err)
	}

	return db
}

/*
查询单行
*/
func queryRowDemo(db *sql.DB) {
	var u user
	// 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放
	err := db.QueryRow("select ACCOUNT_ID,DISPLAY_NAME from c_account limit 1").Scan(&u.accountId, &u.displayName)
	if err != nil {
		fmt.Printf("未找到结果", err)
	}
	fmt.Printf("单行查询结果:"+"id:%d name:%s age:%d\n", u.accountId, u.displayName)

}

/*
查询多行
*/
func queryMultiRowDemo(db *sql.DB) {
	// 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放
	rows, err := db.Query("select ACCOUNT_ID,DISPLAY_NAME from c_accou1nt limit 2")
	if err != nil {
		fmt.Printf("未找到结果", err)
	}
	// 非常重要：关闭rows释放持有的数据库链接
	defer rows.Close()
	// 循环读取结果集中的数据
	fmt.Println("查询结果:")
	for rows.Next() {
		var u user
		err := rows.Scan(&u.accountId, &u.displayName)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Println("多行查询结果:"+"id:%d name:%s", u.accountId, u.displayName)
	}
}

/*
插入数据
*/
func insertRowDemo(db *sql.DB) {
	sqlStr := "insert into user(name, age) values (?,?)"
	ret, err := db.Exec(sqlStr, "王五", 38)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	theID, err := ret.LastInsertId() // 新插入数据的id
	if err != nil {
		fmt.Printf("get lastinsert ID failed, err:%v\n", err)
		return
	}
	fmt.Printf("insert success, the id is %d.\n", theID)
}

/*
更新数据
*/
func updateRowDemo(db *sql.DB) {
	sqlStr := "update user set age=? where id = ?"
	ret, err := db.Exec(sqlStr, 39, 3)
	if err != nil {
		fmt.Printf("update failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("update success, affected rows:%d\n", n)
}

/*
删除数据
*/
func deleteRowDemo(db *sql.DB) {
	sqlStr := "delete from user where id = ?"
	ret, err := db.Exec(sqlStr, 3)
	if err != nil {
		fmt.Printf("delete failed, err:%v\n", err)
		return
	}
	n, err := ret.RowsAffected() // 操作影响的行数
	if err != nil {
		fmt.Printf("get RowsAffected failed, err:%v\n", err)
		return
	}
	fmt.Printf("delete success, affected rows:%d\n", n)
}

/*
定义存储结构体
*/
type user struct {
	accountId   int
	displayName string
}

func main() {
	bb := initBb()
	//单行查询
	queryRowDemo(bb)
	//多行查询
	queryMultiRowDemo(bb)
	//插入数据
	insertRowDemo(bb)
	//更新数据 返回影响行数
	updateRowDemo(bb)
	//删除数据 返回影响行数
	deleteRowDemo(bb)
	//如果数据库连接打开记得关闭
	defer bb.Close()
}

```

# MySQL预处理
## `database/sql`中使用下面的`Prepare`方法来实现预处理操作
```
func (db *DB) Prepare(query string) (*Stmt, error)
```
`Prepare`方法会先将sql语句发送给MySQL服务端，返回一个准备好的状态用于之后的查询和命令。返回值可以同时执行多个查询和命令。
## 占位符语法
```

	 数据库	     占位符语法
   	MySQL	     ?
   	PostgreSQL	  $1, $2等
   	SQLite	     ? 和$1
    Oracle	     :name

```
## 案例
```
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

/*
初始化数据库
*/
func initBb1() (db *sql.DB) {
	dsn := "root:password@tcp(127.0.0.1:3306)/cygn?charset=utf8"
	//打开数据库连接
	db, err := sql.Open("mysql", dsn)
	//设置链接数
	db.SetMaxOpenConns(10)
	//设置连接池最大限制链接数
	db.SetMaxIdleConns(100)
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		fmt.Printf("尝试与数据库建立连接失败", err)
	}
	if err != nil {
		fmt.Printf("连接数据库失败", err)
	}

	return db
}

/*
查询单行
*/
func queryRowDemo(db *sql.DB) {
	var u user
	// 非常重要：确保QueryRow之后调用Scan方法，否则持有的数据库链接不会被释放
	err := db.QueryRow("select ACCOUNT_ID,DISPLAY_NAME from c_account limit 1").Scan(&u.accountId, &u.displayName)
	if err != nil {
		fmt.Printf("未找到结果", err)
	}
	fmt.Printf("单行查询结果:"+"id:%d name:%s age:%d\n", u.accountId, u.displayName)

}

// 预处理查询示例
func prepareQueryDemo(db *sql.DB) {
	sqlStr := "select ACCOUNT_ID,DISPLAY_NAME from user where id > ?"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Printf("prepare failed, err:%v\n", err)
		return
	}
	defer stmt.Close()
	rows, err := stmt.Query(0)
	if err != nil {
		fmt.Printf("query failed, err:%v\n", err)
		return
	}
	defer rows.Close()
	// 循环读取结果集中的数据
	for rows.Next() {
		var u user1
		err := rows.Scan(&u.accountId, &u.displayName)
		if err != nil {
			fmt.Printf("scan failed, err:%v\n", err)
			return
		}
		fmt.Printf("id:%d name:%s", &u.accountId, &u.displayName)
	}
}

// 预处理插入示例
func prepareInsertDemo(db *sql.DB) {
	sqlStr := "insert into user(name, age) values (?,?)"
	stmt, err := db.Prepare(sqlStr)
	if err != nil {
		fmt.Printf("prepare failed, err:%v\n", err)
		return
	}
	defer stmt.Close()
	_, err = stmt.Exec("小王子", 18)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	_, err = stmt.Exec("沙河娜扎", 18)
	if err != nil {
		fmt.Printf("insert failed, err:%v\n", err)
		return
	}
	fmt.Println("insert success.")
}

/*
定义存储结构体
*/
type user1 struct {
	accountId   int
	displayName string
}

func main() {
	bb := initBb()
	//预处理查询示例
	prepareQueryDemo(bb)
	//预处理插入示例
	prepareInsertDemo(bb)
	//如果数据库连接打开记得关闭
	defer bb.Close()
}

```

# 事务

## API
```
Go语言中使用以下三个方法实现MySQL中的事务操作。 开始事务

func (db *DB) Begin() (*Tx, error)
提交事务

func (tx *Tx) Commit() error
回滚事务

func (tx *Tx) Rollback() error
```

## 案例
```
package main

import (
	"database/sql"
	"fmt"
)

func main() {
	bb := initBb1()
	//执行事务
	transactionDemo(bb)
}

/*
初始化数据库
*/
func initBb1() (db *sql.DB) {
	dsn := "root:password@tcp(127.0.0.1:3306)/cygn?charset=utf8"
	//打开数据库连接
	db, err := sql.Open("mysql", dsn)
	//设置链接数
	db.SetMaxOpenConns(10)
	//设置连接池最大限制链接数
	db.SetMaxIdleConns(100)
	// 尝试与数据库建立连接（校验dsn是否正确）
	err = db.Ping()
	if err != nil {
		fmt.Printf("尝试与数据库建立连接失败", err)
	}
	if err != nil {
		fmt.Printf("连接数据库失败", err)
	}

	return db
}

// 事务操作示例
func transactionDemo(bb *sql.DB) {
	tx, err := bb.Begin() // 开启事务
	if err != nil {
		if tx != nil {
			tx.Rollback() // 回滚
		}
		fmt.Printf("begin trans failed, err:%v\n", err)
		return
	}
	sqlStr1 := "Update user set age=30 where id=?"
	ret1, err := tx.Exec(sqlStr1, 2)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql1 failed, err:%v\n", err)
		return
	}
	affRow1, err := ret1.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	sqlStr2 := "Update user set age=40 where id=?"
	ret2, err := tx.Exec(sqlStr2, 3)
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec sql2 failed, err:%v\n", err)
		return
	}
	affRow2, err := ret2.RowsAffected()
	if err != nil {
		tx.Rollback() // 回滚
		fmt.Printf("exec ret1.RowsAffected() failed, err:%v\n", err)
		return
	}

	fmt.Println(affRow1, affRow2)
	fmt.Println("事务提交啦...")
	tx.Commit() // 提交事务

	fmt.Println("exec trans success!")
}

```
