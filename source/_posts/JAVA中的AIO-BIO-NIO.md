---
title: 'JAVA中的AIO,BIO,NIO'
date: 2020-04-08 21:32:46
tags: [java,网络模型]
---

# JAVA中的AIO,BIO,NIO


## 概念

```
AIO: 异步非堵塞IO
BIO: 同步堵塞IO
NIO: 同步非堵塞IO
```

<!--more-->

## 实现

### BIO
服务端 创建两个socket 客户端一个socket

堵塞部分：
1. 等待接收连接部分
2. 等待客户端消息
  

```
就是我们普通的socket连接

ServerSocket ss=new ServerSocket(10000);
            ## 1.堵塞 接收连接请求
            Socket socket = ss.accept();
            ## 2.堵塞 读取客户端消息
            DataInputStream input=new DataInputStream(s.getInputStream());
```
 普通的BIO模型： 会在这两步进行堵塞 


```
如果非要实现非堵塞的话： 
1. 可以加入多线程去操作
2. 建立一个list保留socket 每次进行遍历查看是否有新数据进入然后读取


```

## NIO
这边的话就是同步非阻塞IO
在java中实现的话

```
        List<SocketChannel> socketList = new ArrayList<>();
        try {

            //申请内存空间读取 -》读取字节 可以申请堆外空间
            ByteBuffer byteBuffers = ByteBuffer.allocate(1024);

            ServerSocketChannel serverSocket = ServerSocketChannel.open();
            serverSocket.bind(new InetSocketAddress(8081));
            //设置非堵塞模型
            serverSocket.configureBlocking(false);

            while (true) {
                //监听连接
                SocketChannel accept = serverSocket.accept();
                //表示监听不到连接 查看是否有消息进入
                if (accept == null) {
                    socketList.forEach(client -> {
                        try {
                            int read = client.read(byteBuffers);
                            //大于0表示有读取到数据
                            if (read > 0) {
                                byteBuffers.flip();
                                System.out.println(byteBuffers.toString());
                            }
                        } catch (IOException ex) {
                            ex.printStackTrace();
                        }
                    });
                } else {
                    //获取到连接
                    //设置非堵塞
                    accept.configureBlocking(false);
                    //加入list
                    socketList.add(accept);
                    //然后判断是否有消息接收
                    //进行遍历查看
                }


            }


        } catch (IOException e) {
            e.printStackTrace();
        }
```
这边的话是简单的实现了NIO，使用了一个遍历list的操作进行判断是否有消息接入
这样性能不太好


```
一般的话 需要使用操作系统的实现轮询
1. Windows的select函数
2.Linux的epoll函数

流程：
  应用系统------->操作系统（函数轮询）

底层是利用网卡去自动感知是否有数据socket进入 有请求进入后转给JVM 进行后续处理 
这样就不用在代码中轮询查找消息了

像是redis 这类就是使用epoll去实现的

epoll是linux上的IO多路复用的一种实现
```

一般项目中都是使用neety或者mina实现


## BIO和NIO的差别
在不考虑多线程情况下，BIO是无法处理并发的