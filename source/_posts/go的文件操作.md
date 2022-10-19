---
title: go的文件操作
date: 2022-10-19 14:03:24
tags: [go]
---
# go的文件操作

## 打开文件和关闭文件
```
os.Open()函数可以打开一个文件

file.Colse()函数方法可以关闭文件


file.Read() 读取文件字节流

os.OpenFile() 文件写入操作
```

<!--more-->

## 案例 文件打开读取
```
package main

import (
	"fmt"
	"io"
	"os"
)

func main() {
	//打开文件
	file, err := os.Open("./main.go")
	if err != nil {
		fmt.Println("打开文件失败,err", err)
	}

	//指定文件读取长度
	var tmp = make([]byte, 128)

	//循环读取
	for {
		//文件读取 返回读取的字节数和可能的具体错误 如文件末尾会返回 0和io.EOF
		read, err := file.Read(tmp)
		if err == io.EOF {
			fmt.Println("文件读取完毕")
			return
		}
		if err != nil {
			fmt.Println("文件读取失败:", err)
			return
		}
		fmt.Println("读取了%d个字节数据", read)
		fmt.Println(string(tmp))
		if read < 128 {
			return
		}
	}
	//关闭文件 函数结束自动调用defer
	defer file.Close()
}

```

### 文件缓冲流打开读取 bufio读取
```
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
)

func main() {
	//打开文件
	file, err := os.Open("./main.go")
	if err != nil {
		fmt.Println("打开文件失败,err", err)
	}

	//使用缓冲流读取文件
	reader := bufio.NewReader(file)
	for {
		line, err := reader.ReadSlice('\n')
		if err == io.EOF {
			fmt.Println("文件读取完毕")
			return
		}
		if err != nil {
			fmt.Println("文件读取失败:", err)
			return
		}
		fmt.Println(line)

	}
	//关闭文件 函数结束自动调用defer
	defer file.Close()
}

```

### 使用ioutil工具类读取文件流
```
package main

import (
	"fmt"
	"io/ioutil"
	"os"
)

func main() {
	//使用ioutil打开文件
	content, err := ioutil.ReadFile("./main.go")
	if err != nil {
		fmt.Println("打开文件失败,err", err)
		return
	}
	fmt.Println(string(content))

	//新版本API
	content, err1 := os.ReadFile("./main.go")
	if err1 != nil {
		fmt.Println("打开文件失败,err", err)
		return
	}
	fmt.Println(string(content))

}

```

## 文件写入操作

### os.OpenFile() 文件写入操作
```
func OpenFile(name string,flag int, perm FileMode)(*File, error){
    ...
}

其中:
name: 要打开的文件名

flag: 打开文件的模式.模式有以下几种
os.O_WRONLY  只写
os.O_CREATE  创建文件
os.O_RDONLY  只读
os.O_RDWR    读写
os.O_TRUNC   清空
os.O_APPEND  追加

```

### Write和WriteString
```
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	fileObj, err := os.OpenFile("./main.go", os.O_APPEND, 0666)
	if err != nil {
		fmt.Println("打开文件失败,err", err)
		return
	}
	//写入
	writer := bufio.NewWriter(fileObj)
	writer.Write([]byte("输入的内容"))
	writer.WriteString("输入的字符串")
	//将缓存中的内容写入文件
	writer.Flush()
	defer fileObj.Close()

}

```

### ioutil.writeFile
```
package main

import (
	"fmt"
	"io/ioutil"
)

func main() {
	str := "输入的字符串"
	err := ioutil.WriteFile("./xx.txt", []byte(str), 0666)
	if err != nil {
		fmt.Println("打开文件失败,err", err)
		return
	}
}

```
