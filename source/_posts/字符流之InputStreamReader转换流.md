---
title: 字符流之InputStreamReader转换流
date: 2018-12-03 18:05:52
tags: [IO流]
---
# 参考
[Java_io体系之OutputStreamWriter、InputStreamReader简介、走进源码及示例——17](https://blog.csdn.net/crave_shy/article/details/17239313)

# 转换流 

# OutputStreamWriter 

 输入字符转换流、是输入字节流转向输入字符流的桥梁、用于将输入字节流转换成输入字符流、通过指定的或者默认的编码将从底层读取的字节转换成字符返回到程序中、与OutputStreamWriter一样、本质也是使用其内部的一个类来完成所有工作：StreamDecoder、使用默认或者指定的编码将字节转换成字符、InputStreamReader只是对StreamDecoder进行了封装、isr内部所有方法核心都是调用StreamDecoder来完成的、InputStreamReader只是对StreamDecoder进行了封装、使得我们可以直接使用读取方法、而不用关心内部实现。

<!--more-->

## API

### 构造方法

```
	OutputStreamWriter(OutputStream out, String charsetName)	创建使用指定字符集的 OutputStreamWriter。
	
	OutputStreamWriter(OutputStream out, Charset cs)	 创建使用给定字符集的 OutputStreamWriter。	
	
	OutputStreamWriter(OutputStream out)	 创建使用默认字符编码的 OutputStreamWriter。
	
	OutputStreamWriter(OutputStream out, CharsetEncoder enc)	创建使用给定字符集编码器的 OutputStreamWriter。

```

### 一般方法

```
	String getEncoding()	 返回此流使用的字符编码的名称。	
	
	void write(int c)	写入单个字符。
	
	void write(char cbuf[], int off, int len)	 写入字符数组的某一部分。
	
	void write(String str, int off, int len)	  写入字符串的某一部分。
	
	void flush()	刷新该流的缓冲。
	
	void close()	关闭此流，但要先刷新它。

```

### 源码分析

```
package com.chy.io.original.code;
 
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.nio.charset.Charset;
import java.nio.charset.CharsetEncoder;
 
 
/**
 * 字符输出流、是使用指定的编码或者时系统默认的编码将字节转换成字符。
 * 是OutputStream转换成Writer的桥梁、也是PrintStream、能够在构造的时候指定编码的关键。
 * 实际上是将OutputStream传递给了StreamEncoder来操作out
 */
 
public class OutputStreamWriter extends Writer {
 
	//通过StreamEncoder se进行字节与字符的转码。
    private final StreamEncoder se;
 
    /**
     * 用指定的编码将OutputStream out转换成OutputStreamWriter。
     */
    public OutputStreamWriter(OutputStream out, String charsetName)
	throws UnsupportedEncodingException
    {
    	// 调用父类Writer的构造方法创建一个新的字符流 writer，其关键部分将同步给自身。
		super(out);
		if (charsetName == null)
		    throw new NullPointerException("charsetName");
		//初始化StreamEncoder se.
		se = StreamEncoder.forOutputStreamWriter(out, this, charsetName);
	}
 
    /**
     * 使用默认编码创建osw。
     */
    public OutputStreamWriter(OutputStream out) {
	super(out);
	try {
	    se = StreamEncoder.forOutputStreamWriter(out, this, (String)null);
	} catch (UnsupportedEncodingException e) {
	    throw new Error(e);
        }
    }
 
    /**
     * 使用指定的字符集创建osw
     */
    public OutputStreamWriter(OutputStream out, Charset cs) {
	super(out);
	if (cs == null)
	    throw new NullPointerException("charset");
	se = StreamEncoder.forOutputStreamWriter(out, this, cs);
    }
 
    /**
     * 使用指定的字符编码创建osw
     */
    public OutputStreamWriter(OutputStream out, CharsetEncoder enc) {
	super(out);
	if (enc == null)
	    throw new NullPointerException("charset encoder");
	se = StreamEncoder.forOutputStreamWriter(out, this, enc);
    }
 
    /**
     * 获取字符流使用的字符集、或者编码
     */
    public String getEncoding() {
    	return se.getEncoding();
    }
 
    /**
     * 将osw中的字符flush到底层OutputStream中、此方法不包含flush底层out中的字节
     */
    void flushBuffer() throws IOException {
    	se.flushBuffer();
    }
 
    /**
     * 将字节转换成字符后、向out中写入一个字符（字符的编码在构造OutputStreamWriter时指定或者使用默认编码
     */
    public void write(int c) throws IOException {
    	se.write(c);
    }
 
    /**
     * 将cbuf中的一部分写入到out中
     */
    public void write(char cbuf[], int off, int len) throws IOException {
    	se.write(cbuf, off, len);
    }
 
    /**
     * 将str一部分写入到out中
     */
    public void write(String str, int off, int len) throws IOException {
    	se.write(str, off, len);
    }
 
    /**
     * flush此流、实际上是调用StreamEncoder的flush方法、
     */
    public void flush() throws IOException {
    	se.flush();
    }
 
    /**
     * 关闭StreamEncoder、即关闭OutputSteramWriter.
     */
    public void close() throws IOException {
    	se.close();
    }
}

```

## InputStreamReader

### 构造方法

```
	InputStreamReader(InputStream in)	创建一个使用默认字符集的 InputStreamReader。
	
	InputStreamReader(InputStream in, String charsetName)	创建使用指定字符集的 InputStreamReader。
	
	InputStreamReader(InputStream in, Charset cs)	 创建使用给定字符集的 InputStreamReader。
	
	InputStreamReader(InputStream in, CharsetDecoder dec)	创建使用给定字符集解码器的 InputStreamReader。

```
### 普通方法

```
	String getEncoding()	返回此流使用的字符编码的名称。
	
	int read()	 读取单个字符。
	
	int read(char cbuf[], int offset, int length)	将字符读入数组中的某一部分。
	
	boolean ready()		判断此流是否已经准备好用于读取。
	
	void close()	关闭该流并释放与之关联的所有资源。

```

### 源码分析

```
package com.chy.io.original.code;
 
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.nio.charset.Charset;
import java.nio.charset.CharsetDecoder;
 
 
 
/**
 * 字符输入流、是连接字节流与字符流之间的桥梁。将读取的字节转换成字符、编码可以使用系统默认的也可以使用指定的。
 * 为提高效率、一般结合BufferedReader使用。
 *
 * @version 	1.1, 13/11/16
 * @author		andyChen
 */
 
public class InputStreamReader extends Reader {
	//可以使用指定编码读取字节的本质。使用其将byte解码成字符串返回。
	//对应的OutputStreamWriter中的StreamEncoder将字节编码成字符写入out中。
	//对使用OutputStreamWriter写入的out读取时、注意编码。否则会造成乱码。
    private final StreamDecoder sd;
 
    /**
     * 使用默认编码将字节输入流InputStream in转换成字符输入流
     */
    public InputStreamReader(InputStream in) {
	super(in);
        try {
	    sd = StreamDecoder.forInputStreamReader(in, this, (String)null); // ## check lock object
        } catch (UnsupportedEncodingException e) {
	    // The default encoding should always be available
	    throw new Error(e);
	}
    }
 
    /**
     * 使用指定的字符集将in转换成isr.
     */
    public InputStreamReader(InputStream in, Charset cs) {
        super(in);
	if (cs == null)
	    throw new NullPointerException("charset");
	sd = StreamDecoder.forInputStreamReader(in, this, cs);
    }
 
    /**
     * 使用指定的字符编码将in转换成isr
     */
    public InputStreamReader(InputStream in, CharsetDecoder dec) {
        super(in);
		if (dec == null)
		    throw new NullPointerException("charset decoder");
		sd = StreamDecoder.forInputStreamReader(in, this, dec);
    }
 
    /**
     * 获取此流的编码
     */
    public String getEncoding() {
    	return sd.getEncoding();
    }
    
    //读取单个字符
    public int read() throws IOException {
        return sd.read();
    }
 
    //将字符读取到cbuf中
    public int read(char cbuf[], int offset, int length) throws IOException {
    	return sd.read(cbuf, offset, length);
    }
 
    //查看此流是否可读
    public boolean ready() throws IOException {
    	return sd.ready();
    }
 
    //关闭此流释放资源
    public void close() throws IOException {
    	sd.close();
    }
}

```