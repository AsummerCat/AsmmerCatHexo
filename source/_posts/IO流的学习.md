---
title: IO流的学习
date: 2018-12-03 10:45:33
tags: IO流
---

# [Demo地址](https://github.com/AsummerCat/ioDemo)

# IO流

关于java IO流的操作是非常常见的，基本上每个项目都会用到，每次遇到都是去网上找一找就行了，屡试不爽。

虽然也能找到，但自己总感觉不是很踏实.

IO流对象的继承关系(如下图)：

![IO流结构图](/img/2018-12-2/IO流.png)

<!--more-->

# 字符流的由来

 因为数据编码的不同，而有了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。 字节流和字符流的区别：   

- 读写单位不同：字节流以字节（8bit）为单位，字符流以字符为单位，根据码表映射字符，一次可能读多个字节。
- 处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型的数据。

##  基本分为两类 

 字节流可用于任何类型的对象，包括二进制对象，而字符流只能处理字符或者字符串

* 字符流 :Reader和 Writer.两个是为字符流（一个字符占两个字节）设计的,主要用来处理字符或字符串.
* 字节流:InputStream 和OutputStream,两个是为字节流设计的,主要用来处理字节或二进制对象
* 为顶级类

## 结论

只要是处理纯文本数据，就优先考虑使用字符流。 除此之外都使用字节流。



## 字节流与字符流的区别

字节流和字符流使用是非常相似的，那么除了操作代码的不同之外，还有哪些不同呢？

字节流在操作的时候本身是不会用到缓冲区（内存）的，是与文件本身直接操作的，而字符流在操作的时候是使用到缓冲区的

需要注意:==字节流在操作文件时，即使不关闭资源（close方法），文件也能输出，但是如果字符流不使用close方法的话，则不会输出任何内容，说明字符流用的是缓冲区，并且可以使用flush方法强制进行刷新缓冲区，这时才能在不close的情况下输出内容==

---

# 字节流

## InputStream 顶级抽象字节输入类 

```java
 * @author  Arthur van Hoff
 * @see     java.io.BufferedInputStream
 * @see     java.io.ByteArrayInputStream
 * @see     java.io.DataInputStream
 * @see     java.io.FilterInputStream
 * @see     java.io.InputStream#read()
 * @see     java.io.OutputStream
 * @see     java.io.PushbackInputStream
 * @since   JDK1.0
```

##  OutputStream 顶级抽象字节输出类

```java
 * @author  Arthur van Hoff
 * @see     java.io.BufferedOutputStream
 * @see     java.io.ByteArrayOutputStream
 * @see     java.io.DataOutputStream
 * @see     java.io.FilterOutputStream
 * @see     java.io.InputStream
 * @see     java.io.OutputStream#write(int)
 * @since   JDK1.0
 */
```

## 最后必须关闭流

​    Objects.requireNonNull(out).close();

```java
finally {
            //最终关闭流
            Objects.requireNonNull(out).close();
            Objects.requireNonNull(in).close();
        }
```

## FilterInputStream 与 FilterOutputStream  过滤流

FilterOutputStream 的作用是用来“封装其它的输出流，并为它们提供额外的功能”。它主要包括BufferedOutputStream, DataOutputStream和PrintStream。

  * FilterInputStream/FilterOutputStream都是抽象类。
  * FilterInputStream有三个子类：BufferedInputStream、DataInputStream、PushbackInputStream
  * FilterOutputStream也有三个子类：BufferedOutputStream、DataOutoutStream、PrintStream

(01) BufferedOutputStream的作用就是为“输出流提供缓冲功能”。  
(02) DataOutputStream 是用来装饰其它输出流，将DataOutputStream和DataInputStream输入流配合使用，“允许应用程序以与机器无关方式从底层输入流中读写基本 Java 数据类型”。  
(03) PrintStream 是用来装饰其它输出流。它能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。     

以上内容是 顶级字节流的介绍

---

# 字节流基础读写操作

```java
 int len;
            //设置每次读取字节的大小  (如果不设置每次读取字节大小 效率可能会下降)
            byte[] buf = new byte[1024];
            while ((len = in.read(buf)) != -1) {
                out.write(buf, 0, len);
            }
```

以下内容为实现:

## 文件输入输出流 FileInputStream、FileOutputStream 

	FileInputStream(File file); 	通过打开一个到实际文件的连接来创建一个文件字节输入流、该文件通过文件系统中的File对象file指定
	
	FileInputStream(String file);	通过打开一个到实际文件的连接来创建一个文件字节输入流、该文件通过文件系统中的路径名name来指定
	
	FileInputStream((FileDescriptor fdObj)	通过使用文件描述符fdObj创建一个FileInputStream、该文件描述符表示到文件系统中某个实际文件的现有连接


## 缓冲输入输出流 BufferedInputStream、BufferedOutputStream

```java
 public
class BufferedInputStream extends FilterInputStream {
 public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }
  }
    
```

可以用来包装 基础流 比如

```java
 InputStream in = new BufferedInputStream(new FileInputStream(file));
```

```java
public
class BufferedOutputStream extends FilterOutputStream { 
public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
    }
}
```

​	也是用来包装基础流 比如

```java
  OutputStream out =  out = new BufferedOutputStream(new FileOutputStream(file2));
```

需要注意的是 

```java
//默认缓冲区大小是8K 强制输出缓冲  在使用缓冲流的时候使用
out.flush();
//这样可以保证输出的时候缓冲区中没有缓存的数据
```

----



## 数据字节输出流 DataInputStream、DataOutputStream  (个人使用比较少)

将java中的基础数据类型写入数据字节输出流中、保存在存储介质中、然后可以用DataInputStream从存储介质中读取到程序中还原成java基础类型。

```
就是能还原java中的基础类类型

```

##  管道输出流  PipedInputStream、PipedOutputStream 

管道用来把程序、线程或程序块的输出连接到另一个程序、线程或者程序块作为它的输入。

```java
管道输入/输出流可以用两种方式进行连接，一种方式是使用connect()方法：
    PipedInputStream pis  = new PipedInputStream();
    PipedOutputStream pos = new PipedOutputStream();
    pis.connect(pos);
    pos.connect(pis);
     另外一种是直接使用构造方法进行连接：
   PipedInputStream pis = new PipedInputStream();
   PipedOutputStream pos = new PipedOutputStream(pis);
```

  demo

```java
public static void main(String[] args) throws IOException {
        int ch1 = 0;
        PipedInputStream pis = new PipedInputStream();
        PipedOutputStream pos = new PipedOutputStream(pis);
        //也可以使用connect()方法
        //    PipedOutputStream pos = new PipedOutputStream();
        //        pos.connect(pis);
        System.out.println("请输入一个字符，按#结束程序！");
        while ((ch1 = System.in.read()) != '#') {
            pos.write(ch1);
            System.out.println((char) pis.read());
        }
    }
```

----



#  字符流

在程序中一个字符等于两个字节，那么java提供了Reader、Writer两个专门操作字符流的类。

字符输出流：Writer

Writer本身是一个字符流的输出类，此类的定义如下：

public abstract class Writer extends Object implements Appendable，Closeable，Flushable

此类本身也是一个抽象类，如果要使用此类，则肯定要使用其子类，此时如果是向文件中写入内容，所以应该使用FileWriter的子类。

FileWriter类的构造方法定义如下：

public FileWriter(File file)throws IOException

字符流的操作比字节流操作好在一点，就是可以直接输出字符串了，不用再像之前那样进行转换操作了。

## Writer 顶级抽象字符输入类 

```java
 * @see Writer
 * @see   BufferedWriter
 * @see   CharArrayWriter
 * @see   FilterWriter
 * @see   OutputStreamWriter
 * @see     FileWriter
 * @see   PipedWriter
 * @see   PrintWriter
 * @see   StringWriter
```

## Reader 顶级抽象字符输出类

```java
 * @see BufferedReader
 * @see   LineNumberReader
 * @see CharArrayReader
 * @see InputStreamReader
 * @see   FileReader
 * @see FilterReader
 * @see   PushbackReader
 * @see PipedReader
 * @see StringReader
```

## 最后必须关闭流

​    Objects.requireNonNull(out).close();

## 输入必须 冲刷缓冲区

```java
bufferedWriter.flush();
```

## 基础操作



## 文件字符输入输出流 FileWriter,FileReader

```java
//换行
fileWriter.write("\r\n"); 
```

## 缓冲字符流 BufferedReader,BufferedWriter

```java
//换行输出
fileReader.readLine();
```

## 转换流InputStreamReader和OutputStreamWriter 

![转换流](/img/2018-12-2/转换流.png)

 转换流的作用，文本文件在硬盘中以字节流的形式存储时，通过InputStreamReader读取后转化为字符流给程序处理，程序处理的字符流通过OutputStreamWriter转换为字节流保存。

```java
   fis = new FileInputStream(src);
            fos = new FileOutputStream(dest);
            InputStreamReader ir = new InputStreamReader(fis, "UTF-8");
            OutputStreamWriter ow = new OutputStreamWriter(fos, "UTF-8");
```

