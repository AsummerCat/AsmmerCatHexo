---
title: 字节流之FileInputStream
date: 2018-12-03 15:58:07
tags: IO流
---


# 参考[Java_io体系之File、FileInputStream、FileOutputStream简介、走进源码及示例——05](https://blog.csdn.net/crave_shy/article/details/16922777)

# 类功能简介

   文件字节输入流、用于将指定连接的文件中的字节读取到程序中。本质是针对文件进行操作、关键部分是创建实例的时候、会根据传入的参数打开到文件的指定连接、创建此文件字节输入流到文件对象的管道来读取文件中的字节、他的很多的方法都是底层系统自带的方法、会用、并且知道在使用此类操作文件的时候此类内部是调用什么方法、通过怎么样的过程就行

<!--more-->

# FileInputStream

## 构造方法

```java
	FileInputStream(File file); 	通过打开一个到实际文件的连接来创建一个文件字节输入流、该文件通过文件系统中的File对象file指定
	
	FileInputStream(String file);	通过打开一个到实际文件的连接来创建一个文件字节输入流、该文件通过文件系统中的路径名name来指定
	
	FileInputStream((FileDescriptor fdObj)	通过使用文件描述符fdObj创建一个FileInputStream、该文件描述符表示到文件系统中某个实际文件的现有连接

```

## 一般方法 



```java
	int avaliable();	返回连接文件中的可供读取的字节数
	
	void close();		关闭fis、并释放所有与此流有关的资源
	
	int read();			读取文件中的一个字节并以整数形式返回
	
	int read(byte[] b, int off, int len)将文件中len个字节读取到下标从off开始的字节数组b中
	
	int read(byte[] b) 读取b.length个字节到字节数组b中
	
	long skip(long n);	跳过文件中n个字节

```

##  源码分析



```java
/**
 * 关于File的输入输出流中的关键实现方法都是系统方法、所以对于fis、fos的使用、知道中间的过程就行。 
 */
 
 
public class FileInputStream extends InputStream {
    java.io.FileInputStream
    //一个打开到指定文件的连接或者句柄。
    private FileDescriptor fd;
 
    //文件夹通道
    private FileChannel channel = null;
    /**
     * 通过“文件路径名”创建一个对应的“文件输入流”。
     */
    public FileInputStream(String name) throws FileNotFoundException {
        this(name != null ? new File(name) : null);
    }
 
    /**
     * 通过“文件对象”创建一个对应的“文件输入流”。
     */
    public FileInputStream(File file) throws FileNotFoundException {
       String name = (file != null ? file.getPath() : null);
       SecurityManager security = System.getSecurityManager();
       if (security != null) {
           security.checkRead(name);
       }
          if (name == null) {
                throw new NullPointerException();
          }
       fd = new FileDescriptor();
       //打开到文件名指定的文件的连接、要对文件进行操作、当然要先获得连接。
       open(name);
    }
 
    /**
     * 通过“文件描述符”创建一个对应的“文件输入流”。
     * 很少用！
     */
    public FileInputStream(FileDescriptor fdObj) {
       SecurityManager security = System.getSecurityManager();
       if (fdObj == null) {
           throw new NullPointerException();
       }
       if (security != null) {
           security.checkRead(fdObj);
       }
       fd = fdObj;
   }
 
    /**
     * 通过文件名打开到“文件名指定的文件”的连接
     */
    private native void open(String name) throws FileNotFoundException;
 
    /**
     * 底层————读取“指定文件”下一个字节并返回、若读取到结尾则返回-1。
     */
    public native int read() throws IOException;
 
 
    /**
     * 底层————读取“指定文件”最多len个有效字节、并存放到下标从 off 开始长度为 len 的 byte[] b 中、返回读取的字节数、若读取到结尾则返回-1。
     */
    private native int readBytes(byte b[], int off, int len) throws IOException;
 
    /**
     * 调用底层readbytes(b, 0, b.length)读取b.length个字节放入b中、返回读取的字节数、若读取到结尾则返回-1。
     */
    public int read(byte b[]) throws IOException {
    	return readBytes(b, 0, b.length);
    }
 
    /**
     * 调用底层readbytes(b, off, len)读取b.length个字节放入下标从 off 开始长度为 len 的 byte[] b 中、返回读取的字节数、若读取到结尾则返回-1。
     */
    public int read(byte b[], int off, int len) throws IOException {
    	return readBytes(b, off, len);
    }
 
    /**
     * 底层————从源文件中跳过n个字节、n不能为负、返回值是实际跳过的字节数。
     * 当n >= fis.available();时返回-1;                  
     */
    public native long skip(long n) throws IOException;
 
    /**
     * 查询此文件字节输入流对应的源文件中可被读取的有效字节数。        
     */
    public native int available() throws IOException;
 
    /**
     * 关闭此流、并释放所有与此流有关的资源、
     * 并且根据此流建立的channel也会被关闭。
     */
    public void close() throws IOException {
        if (channel != null)
            channel.close();
        close0();
    }
 
    /**
     * 获取此流对应的文件系统中实际文件的“文件描述符”。
     */
    public final FileDescriptor getFD() throws IOException {
       if (fd != null) return fd;
       throw new IOException();
    }
 
    /**
     * 获取与此流有关的FileChannel。
    */ 
    public FileChannel getChannel() {
       synchronized (this) {
          if (channel == null)
          channel = FileChannelImpl.open(fd, true, false, this);
           return channel;
       }
    }
     
    private static native void initIDs();
    //关闭
    private native void close0() throws IOException;
 
    static {
    	initIDs();
    }
 
    /**
     * 当此流不再被使用、关闭此流。
     */
   protected void finalize() throws IOException {
       if (fd != null) {
          if (fd != fd.in) {
          	close();
           }
       }
   }
}

```

# FileOutputStream



## 构造方法

```java
	FileOutputStream(File file)		创建一个向指定File表示的文件中写入数据的文件字节输出流 fos、每次写入会替换掉File中原有内容
	
	FileOutputStream(File file, boolean append)		创建一个向指定File表示的文件中写入数据的 fos、每次写入会根据append的值判断是否会在File中原有内容之后追加写入的内容。
	
	FileOutputStream(String name)	创建一个向name指定的文件中写入数据的fos、每次写入会替换掉File中原有内容
	
	FileOutputStream(String name, boolean append)	创建一个向name指定的文件中写入数据的fos、每次写入会根据append的值判断是否会在File中原有内容之后追加写入的内容。 append默认为false
	
	FileOutputStream(FileDescriptor fdObj)		创建一个向指定文件描述符处写入数据的输入流fos、该文件描述符表示一个到文件系统中的某个实际文件的现有连接。

```

## 一般方法

```java
	void close(); 	关闭当前流、释放与此流有关的所有资源
	
	void finalize();	清理到文件的连接、并确保不再使用此流的时候关闭此流
	
	FileChannel getChannel();	返回与此文件流唯一有关的FileChannel
	
	FileDescriptor getFD();		返回与此流有关的文件描述符
	
	void write(byte b);			将一个字节写入到fos中
	
	void write(byte[] b);		将一个字节数组写入到fos中
	
	void write(byte[] b,int off, int len)	将字节数组b的一部分写入到fos中
```

## 源码分析



```java
public class FileOutputStream extends OutputStream {

    private FileDescriptor fd;
 
    private FileChannel channel= null;
 
    //用于标识写入的内容是追加还是替换。
    private boolean append = false;
 
    /**
     *根据具体文件名打开一个文件字节输出流。name中表示的文件夹必须存在、文件可以自动创建
     *比如String name = "D:\\directory1\\directory2\\fos.txt"; 则在D盘中必须有个目录
     *名为directory1且还有下一层目录directory2。至于是否有fos.txt没有关系、有：fos直接写入
     *没有：fos写入之前会创建一个新的。
     *实质是调用下面的 public FileOutputStream(File file)来创建fos;
     */
    public FileOutputStream(String name) throws FileNotFoundException {
    	this(name != null ? new File(name) : null, false);
    }
 
    /**
     * 同上面一个构造方法、只是比上面一个构造方法多了一个参数 boolean append、
     * 1）若append 值为  true 则表示保留原文件中数据、在其后面添加新的数据。
     * 2）若append 值为false 如果文件存在、则清空文件内容、如果文件不存在则创建新的。
     * 也可看出、上面一个构造方法就是默认append为false的这个构造方法的特例。
     * 实质是调用下面的public FileOutputStream(File file, boolean append)来创建fos;
     */
    public FileOutputStream(String name, boolean append)
        throws FileNotFoundException
    {
        this(name != null ? new File(name) : null, append);
    }
 
    /**
     * 根据指定的File来创建一个到此文件的文件字节输出流。
     */
    public FileOutputStream(File file) throws FileNotFoundException {
    	this(file, false);
    }
 
    /**
     * 根据指定的File来创建一个到此文件的文件字节输出流。可以指定是追加还是替换。
     */
    public FileOutputStream(File file, boolean append)
        throws FileNotFoundException
    {
       String name = (file != null ? file.getPath() : null);
       SecurityManager security = System.getSecurityManager();
       if (security != null) {
           security.checkWrite(name);
       }
           if (name == null) {
               throw new NullPointerException();
           }
       fd = new FileDescriptor();
         this.append = append;
        //调用底层方法真正的建立到指定文件的字节输出流。   
       if (append) {
          openAppend(name);
       } else {
          open(name);
       }
    }
 
    /**
     * 通过文件描述符打开一个到实际文件的文件字节输出流。
     */
    public FileOutputStream(FileDescriptor fdObj) {
       SecurityManager security = System.getSecurityManager();
       if (fdObj == null) {
          throw new NullPointerException();
       }
       if (security != null) {
           security.checkWrite(fdObj);
       }
       fd = fdObj;
    }
 
    /**
     * 底层————打开一个准备写入字节的文件、模式为替换。
     */
    private native void open(String name) throws FileNotFoundException;
 
    /**
     * 底层————打开一个准备写入字节的文件、模式为追加。
     */
    private native void openAppend(String name) throws FileNotFoundException;
 
    /**
     * 底层————向文件中写入一个字节
     */
    public native void write(int b) throws IOException;
 
    /**
     * 底层————将一个起始下标为  off 长度为len的字节数组 b 文件中
     */
    private native void writeBytes(byte b[], int off, int len) throws IOException;
 
    /**
     *底层————将整个字节数组 b 文件中
     */
    public void write(byte b[]) throws IOException {
    	writeBytes(b, 0, b.length);
    }
 
    /**
     * 调用底层writerBytes(byte b, int off, int len)将一个起始下标为  off 长度为len的字节数组 b 文件中
     */
    public void write(byte b[], int off, int len) throws IOException {
    	writeBytes(b, off, len);
    }
 
    /**
     * 关闭此流、并释放所有与此流有关的资源。同时将建立在此流基础上的FileChannel也关闭。
     */
    public void close() throws IOException {
        if (channel != null)
            channel.close();
        close0();
    }
 
    /**
     * 获得此流对应的文件描述符。
     */
     public final FileDescriptor getFD()  throws IOException {
       if (fd != null) return fd;
       throw new IOException();
     }
    
    /**
     * 获得与此流相关的FileChannel。
     
    public FileChannel getChannel() {
		synchronized (this) {
		    if (channel == null)
				channel = FileChannelImpl.open(fd, false, true, this, append);
			    return channel;
		}
    }
     */
    /**
     *确保此流不再被引用时、将此流中的数据flush到指定文件中并且关闭此流。
     */
    @SuppressWarnings("static-access")
    protected void finalize() throws IOException {
 	if (fd != null) {
 	    if (fd == fd.out || fd == fd.err) {
 	    	//调用父类OutputStream的flush();
 	    	flush();
 	    } else {
 	    	close();
 	    }
 	}
    }
 
    private native void close0() throws IOException;
 
    private static native void initIDs();
    
    static {
    	initIDs();
    }
}
```

# 总结

```java
 FileInputStream（fis）、FileOutputStream（fos）、本质是和文件打交道、两者在构造方法中都会通过传入的File对象或者表示File对象的路径名来创建实例、创建实

例的过程中会根据不同的需求打开到指定文件对象的连接（核心是open(String filename)、openAppend(String filename)追加模式）方法、并且一般情况下类中的对单个字节

操作的方法read()或者write()方法是读写的核心、但是此两类中不是、他们的读/写字节、读/写字节数组、是分别实现的、都是用系统的方法实现的、其中                       

read(byte[] b, int off, int len)和write(byte[] b, intoff, int len)是调用自己本身私有方法readBytes(byte[] b, int off, int len) 和write(byte[] b, intoff, int len)来实现的、而非循环调用read()、write()。
```