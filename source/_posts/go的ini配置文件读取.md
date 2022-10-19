---
title: go的ini配置文件读取
date: 2022-10-19 14:04:24
tags: [go]
---
# go的ini配置文件读取

ps: 也可以直接github 上搜索 go ini 直接引用现成三方库

## 例如
```
conf.ini

内容:
[mysql]
address=10.10.10.10
port=3306
username=root
password=root

[redis]
address=10.10.10.10
port=6379
username=
password=
db=1
```
<!--more-->

### 1.首先建立myconfig.ini文件，并写入以下内容
```
[mysql]
address=localhost
port=3306
username=root
password=root

```
### 2.建立对应的结构体
```
//这个里面的方法和成员变量一定要大写，否则访问不到
type MysqlConfig struct {
	Address  string	`ini:"address"`
	Port     int	`ini:"port"`
	Username string	`ini:"username"`
	Password string	`ini:"password"`
}
//如果有多个配置可以新建多个建构体

```

### 3. 新建配置类的所有结构体（整合结构体）
```
type config struct {
	//这个结构体的意义是：例如我们在找到配置文件的[mysql]，就知道我应该给mysql的结构体赋值。而不是其他的结构体。通过`ini:"mysql"`能找到我要操作的结构体的名称是MysqlConfig 
	MysqlConfig `ini:"mysql"`
	//这里是为了适应多个配置的情况。暂时就只写一个了
	//RedisConfig `ini:"redis"`
}

```

### 4.加载配置类
```
//这里的data传的应该是&config ，是为了做到多配置的兼容。
func loadIni(fileName string, data interface{}) (err error) {

	t := reflect.TypeOf(data)
	//判断是指针类型
	if t.Kind() != reflect.Ptr {
		err = fmt.Errorf("no ptr")
	}
	//判断必须是结构体指针
	if t.Elem().Kind() !=reflect.Struct{
		err = fmt.Errorf("no struct")
	}

	file, err := ioutil.ReadFile(fileName)
	if err != nil {
		return
	}
	//以行切割
	split := strings.Split(string(file), "\r\n")
	fmt.Println(split)
	var structName string
	for index, line := range split {
		if len(line) == 0{
			continue
		}
		//去掉空格
		line = strings.TrimSpace(line)
		//fmt.Println(data)
		//如果是注释就跳过
		if strings.HasPrefix(line,";") || strings.HasPrefix(line,"#"){
			continue
		}
		//先找要操作的结构体
		if strings.HasPrefix(line,"["){

			//如果是[]这类的就是section
			if line[0] !='[' || line[len(line)-1] != ']'{
				fmt.Errorf("%d line error",index +1 )
				return
			}
			sectioName := strings.TrimSpace(line[1:len(line)-1])
			if len(sectioName) == 0{
				fmt.Errorf("%d line error",index +1 )
				return
			}
			//反射之后遍历config的成员变量，找到要操做的结构体
			for i := 0; i < t.Elem().NumField(); i++ {

				field := t.Elem().Field(i)
				if sectioName == field.Tag.Get("ini"){
					//找到了对应的字段名
					structName = field.Name
					fmt.Printf("找到了%s的结构体,%s\n",sectioName,structName)
					break
				}
			}
		}else {
			//再找键值对
			//如果是键值对，以等号分割去data里面取出结构体
			//遍历嵌套结构体
			if strings.Index(line,"=") == -1 || strings.HasPrefix(line,"=") {
				err = fmt.Errorf("DATA:%s error" ,index +1)
			}

			index := strings.Index(line, "=")
			key :=strings.TrimSpace(line[:index])
			value :=strings.TrimSpace(line[index+1:])
			v := reflect.ValueOf(data)
			fmt.Printf("key ===%s\n",key)
			fmt.Printf("value ===%s\n",value)
			//拿到结构体Value值
			sValue:= v.Elem().FieldByName(structName)
			sType := sValue.Type()
			
			if sType.Kind() != reflect.Struct{
				fmt.Errorf("不是结构体")
			}
			var fileName string
			var fileType reflect.StructField
			//找到结构体的某个字段
			for i := 0; i < sValue.NumField(); i++ {
				filed := sType.Field(i)//tag是存储再类型信息中的
				fileType = filed
				if filed.Tag.Get("ini") == key {
					fileName = filed.Name
					fmt.Printf("fileName ===%s\n",fileName)
					break
				}
			}
			//错误字段，直接跳出。其实应该报错的。
			if len(fileName)== 0{
				continue
			}
			//通过key值找到了某个成员变量的名称，拿到这个成员变量的操作对象
			fileObj := sValue.FieldByName(fileName)
			//根据类型来赋值在配置文件中拿到的value
			switch fileType.Type.Kind() {
			//判断并赋值
			case reflect.String:
				fileObj.SetString(value)
			case reflect.Int,reflect.Int8,reflect.Int16,reflect.Int32,reflect.Int64:
				parseInt, err := strconv.ParseInt(value, 10, 64)
				if err != nil {
					err = fmt.Errorf("转换错误")
				}
				fileObj.SetInt(parseInt)
				//这里还可以添加其他的类型。
			}
		}
	}
	return

}

```

### 5.测试
```
func main() {
	var mc config
	//传入配置文件的位置。和config结构体的地址
	err := loadIni("./ini/myconfig.ini", &mc)
	if err != nil {
		panic(err)
	}
	fmt.Println("看看反射后的结果：",mc.MysqlConfig)
}

```


# 完整代码
```
package main

import (
	"fmt"
	"io/ioutil"
	"reflect"
	"strconv"
	"strings"
)

//ini配置文件解析器

// MsqlConf 结构体
// 这个里面的方法和成员变量一定要大写，否则访问不到
type MysqlConfig struct {
	Address  string `ini:"address"`
	Port     int    `ini:"port"`
	Username string `ini:"username"`
	Password string `ini:"password"`
}

//如果有多个配置可以新建多个建构体

// 新建配置类的所有结构体（整合结构体）
type config struct {
	//这个结构体的意义是：例如我们在找到配置文件的[mysql]，就知道我应该给mysql的结构体赋值。而不是其他的结构体。通过`ini:"mysql"`能找到我要操作的结构体的名称是MysqlConfig
	MysqlConfig `ini:"mysql"`
	//这里是为了适应多个配置的情况。暂时就只写一个了
	//RedisConfig `ini:"redis"`
}

// 这里的data传的应该是&config ，是为了做到多配置的兼容。
func loadIni(fileName string, data interface{}) (err error) {

	t := reflect.TypeOf(data)
	//判断是指针类型
	if t.Kind() != reflect.Ptr {
		err = fmt.Errorf("no ptr")
	}
	//判断必须是结构体指针
	if t.Elem().Kind() != reflect.Struct {
		err = fmt.Errorf("no struct")
	}

	file, err := ioutil.ReadFile(fileName)
	if err != nil {
		return
	}
	//以行切割
	split := strings.Split(string(file), "\r\n")
	fmt.Println(split)
	var structName string
	for index, line := range split {
		if len(line) == 0 {
			continue
		}
		//去掉空格
		line = strings.TrimSpace(line)
		//fmt.Println(data)
		//如果是注释就跳过
		if strings.HasPrefix(line, ";") || strings.HasPrefix(line, "#") {
			continue
		}
		//先找要操作的结构体
		if strings.HasPrefix(line, "[") {

			//如果是[]这类的就是section
			if line[0] != '[' || line[len(line)-1] != ']' {
				fmt.Errorf("%d line error", index+1)
				return
			}
			sectioName := strings.TrimSpace(line[1 : len(line)-1])
			if len(sectioName) == 0 {
				fmt.Errorf("%d line error", index+1)
				return
			}
			//反射之后遍历config的成员变量，找到要操做的结构体
			for i := 0; i < t.Elem().NumField(); i++ {

				field := t.Elem().Field(i)
				if sectioName == field.Tag.Get("ini") {
					//找到了对应的字段名
					structName = field.Name
					fmt.Printf("找到了%s的结构体,%s\n", sectioName, structName)
					break
				}
			}
		} else {
			//再找键值对
			//如果是键值对，以等号分割去data里面取出结构体
			//遍历嵌套结构体
			if strings.Index(line, "=") == -1 || strings.HasPrefix(line, "=") {
				err = fmt.Errorf("DATA:%s error", index+1)
			}

			index := strings.Index(line, "=")
			key := strings.TrimSpace(line[:index])
			value := strings.TrimSpace(line[index+1:])
			v := reflect.ValueOf(data)
			fmt.Printf("key ===%s\n", key)
			fmt.Printf("value ===%s\n", value)
			//拿到结构体Value值
			sValue := v.Elem().FieldByName(structName)
			sType := sValue.Type()

			if sType.Kind() != reflect.Struct {
				fmt.Errorf("不是结构体")
			}
			var fileName string
			var fileType reflect.StructField
			//找到结构体的某个字段
			for i := 0; i < sValue.NumField(); i++ {
				filed := sType.Field(i) //tag是存储再类型信息中的
				fileType = filed
				if filed.Tag.Get("ini") == key {
					fileName = filed.Name
					fmt.Printf("fileName ===%s\n", fileName)
					break
				}
			}
			//错误字段，直接跳出。其实应该报错的。
			if len(fileName) == 0 {
				continue
			}
			//通过key值找到了某个成员变量的名称，拿到这个成员变量的操作对象
			fileObj := sValue.FieldByName(fileName)
			//根据类型来赋值在配置文件中拿到的value
			switch fileType.Type.Kind() {
			//判断并赋值
			case reflect.String:
				fileObj.SetString(value)
			case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
				parseInt, err := strconv.ParseInt(value, 10, 64)
				if err != nil {
					err = fmt.Errorf("转换错误")
				}
				fileObj.SetInt(parseInt)
				//这里还可以添加其他的类型。
			}
		}
	}
	return

}

func main() {
	var mc config
	//传入配置文件的位置。和config结构体的地址
	err := loadIni("D:\\godemo\\godemo\\day5\\myconfig.ini", &mc)
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("看看反射后的结果：", mc.MysqlConfig)
}

```