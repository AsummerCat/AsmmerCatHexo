---
title: 在线聊天室实现-使用Socket
date: 2019-11-09 14:44:13
tags: [java,WebSocket,socket]
---

# 在线聊天室实现-使用Socket

[demo地址](https://github.com/AsummerCat/im-websocket-demo)

# 读取套接字

```
package com.linjingc.top.imwebsocketdemo.socketService;

import java.io.ByteArrayOutputStream;
import java.io.InputStream;

/**
 * 读取套接字
 * @author cxc
 * @date 2019/11/9 15:59
 */
public class SendMessage {
    public static String readStream(InputStream in) {
        try {
            //<1>创建字节数组输出流，用来输出读取到的内容
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            //<2>创建缓存大小
            byte[] buffer = new byte[1024]; // 1KB
            //每次读取到内容的长度
            int len = -1;
            //<3>开始读取输入流中的内容
            while ((len = in.read(buffer)) != -1) { //当等于-1说明没有数据可以读取了
                baos.write(buffer, 0, len);   //把读取到的内容写到输出流中
            }
            //<4> 把字节数组转换为字符串
            String content = baos.toString();
            //<5>关闭输入流和输出流
            in.close();
            baos.close();
            //<6>返回字符串结果
            return content;
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }

}

```

<!--more-->

# (服务端推送消息)线程

```java
package com.linjingc.top.imwebsocketdemo.socketService;

import java.io.IOException;
import java.io.PrintStream;
import java.net.Socket;
import java.util.Map;
import java.util.Scanner;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class ExecuteClientThread implements Runnable {

    private static Map<String, Socket> clientMap = new ConcurrentHashMap<>();//存储所有的用户信息
    private Socket client;//每一个服务器线程对应一个客户端线程


    ExecuteClientThread(Socket client) {
        this.client = client;
    }

    @Override
    public void run() {
        //防止一个客户端多次注册所做的标记位置
        boolean Flag = true;
        try {
            //服务器向用户输出一些提示信息
            PrintStream PrintToCilent = new PrintStream(client.getOutputStream());


            Scanner scanner = new Scanner(client.getInputStream());
            //用户外部的输入信息
            String str = null;
            while (true) {
                if (scanner.hasNext()) {
                    //外部的用户输出
                    str = scanner.next();
                    //排除特殊符号
                    Pattern pattern = Pattern.compile("\r");
                    Matcher matcher = pattern.matcher(str);
                    str = matcher.replaceAll("");

                    if (str.startsWith("userName")) {
                        String userName = str.split(":")[1];
                        userRegist(userName, client, Flag);
                        Flag = false;
                    }
                    // 群聊流程
                    else if (str.startsWith("G:")) {
                        PrintToCilent.println("已进入群聊模式！");
                        groupChat(scanner, client);
                    }
                    // 私聊流程
                    else if (str.startsWith("P")) {//模式
                        String userName = str.split("-")[1];
                        PrintToCilent.println("已经进入与" + userName + "的私聊");

                        privateChat(scanner, userName);
                    }
                    // 用户退出
                    else if (str.contains("byebye")) {
                        String userName = null;
                        for (String getKey : clientMap.keySet()) {
                            if (clientMap.get(getKey).equals(client)) {
                                userName = getKey;
                            }
                        }

                        System.out.println("用户" + userName + "下线了..");
                        clientMap.remove(userName);//将此实例从map中移除
                    }
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 注册用户
     * @param userName
     * @param client
     * @param Flag
     * @throws IOException
     */
    private void userRegist(String userName, Socket client, boolean Flag) throws IOException {
        //服务器向用户输出一些提示信息
        PrintStream PrintToCilent = new PrintStream(client.getOutputStream());
        if (Flag) {
            System.out.println("用户" + userName + "上线了！");

            //把用户加入储存map
            clientMap.put(userName, client);
            System.out.println("当前群聊人数为" + (clientMap.size()) + "人");
            PrintToCilent.println("注册成功！");
        } else {
            PrintToCilent.println("警告:一个客户端只能注册一个用户！");
        }
    }


    /**
     * 聊天室消息
     * @param scanner
     * @param client
     * @throws IOException
     */
    private void groupChat(Scanner scanner, Socket client) throws IOException {
        // 取出clientMap中所有客户端Socket，然后遍历一遍
        // 分别取得每个Socket的输出流向每个客户端输出
        //在群聊的时候服务器向客户端发送数据
        PrintStream PrintToClient = new PrintStream(client.getOutputStream());
        boolean ExitFlag = false;

        Set<Map.Entry<String, Socket>> entrySet =
                clientMap.entrySet();

        String userName = null;
        //获得:是哪个用户说的话
        for (Map.Entry<String, Socket> socketEntry : entrySet) {
            if (socketEntry.getValue() == client) {
                //发出信息的用户
                userName = socketEntry.getKey();
            }
        }
        String msg = null;

        while (true) {
            if (scanner.hasNext()) {
                msg = scanner.next();
                //如果用户退出了
                if ("exit".equals(msg)) {
                    for (Map.Entry<String, Socket> stringSocketEntry : entrySet) {
                        new PrintStream(stringSocketEntry.getValue().getOutputStream(), true).println("用户" + userName + "刚刚退出了群聊！！");//给所有人发退出群聊的消息
                    }
                    return;
                }

//遍历用户的map，获取所有用户的Socket
                for (Map.Entry<String, Socket> stringSocketEntry : entrySet) {
                    try {
                        Socket socket = stringSocketEntry.getValue();
                        PrintStream ps = new PrintStream(socket.getOutputStream(), true);
                        //给每个用户发消息
                        ps.println("群聊:用户" + userName + "说: " + msg);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }

            }
        }

    }

    /**
     * 私聊消息
     * @param scanner
     * @param privatepeopleName
     * @throws IOException
     */
    private void privateChat(Scanner scanner, String privatepeopleName) throws IOException {

        Socket privateUser = clientMap.get(privatepeopleName);
        //拿到私聊对象的输出流
        PrintStream ps = new PrintStream(privateUser.getOutputStream());
        //拿到当前客户端的输出流
        PrintStream PrintToClient = new PrintStream(client.getOutputStream());
        String Message = null;
        String MyName = null;
        Set<Map.Entry<String, Socket>> set = clientMap.entrySet();
        for (Map.Entry<String, Socket> value : set) {
            if (value.getValue() == client) {
                MyName = value.getKey();
                break;
            }
        }

        while (true) {
            if (scanner.hasNext()) {
                Message = scanner.next();
                //如果用户输入了退出
                if ("exit".equals(Message)) {
                    PrintToClient.println("已退出和" + privatepeopleName + "的私聊");
                    ps.println("对方已经退出了私聊");
                    break;
                }
                //如果用户没有退出，向私聊对象发送消息
                ps.println(MyName + "说" + Message);
            }
        }
    }
}

```



# 服务端

```
package com.linjingc.top.imwebsocketdemo.socketService;

import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author Administrator
 */
public class SocketServiceImpl {


	public static void main(String[] args) {
		try {
			//最多容纳100个客户端聊天
			ExecutorService executorService = Executors.newFixedThreadPool(100);
			//监听6655号端口
			ServerSocket serverSocket = new ServerSocket(6655);
			for (int i = 0; i < 100; i++) {
				Socket client = serverSocket.accept();
				System.out.println("有新的用户连接 " + client.getInetAddress() + client.getPort());
				executorService.execute(new ExecuteClientThread(client));
			}
			executorService.shutdown();
			serverSocket.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}

```



# 客户端

```
package com.linjingc.top.imwebsocketdemo.socketService;

import java.io.IOException;
import java.io.PrintStream;
import java.net.Socket;
import java.util.Scanner;

class ExcuteServerInPut implements Runnable{//接收服务器的数据
    private Socket ToServer;
 
    ExcuteServerInPut(Socket ToServer){
        this.ToServer = ToServer;
    }
 
    @Override
    public void run() {
        try {
            Scanner scanner = new Scanner(ToServer.getInputStream());
               while (scanner.hasNext()){
                System.out.println(scanner.nextLine());
            }
            scanner.close();
            ToServer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
class ExcuteServerOutPut implements Runnable{//向服务器发送数据
 
    private Socket Socket;
    ExcuteServerOutPut(Socket Socket){
        this.Socket = Socket;
    }
 
    @Override
    public void run() {
        try {
            PrintStream printStream = new PrintStream(Socket.getOutputStream());
            Scanner scanner = new Scanner(System.in);
            scanner.useDelimiter("\n");
            System.out.println("*****************************************");
            System.out.println("***用户注册:useerName:同户名(仅限一次)***");
            System.out.println("***进入群聊:G:           退出群聊:exit***");
            System.out.println("***私聊:P-用户名         退出私聊:exit***");
            System.out.println("***********退出聊天室:byebye*************");
            while (true){
                if(scanner.hasNext()) {
                    String string = scanner.next();
                    printStream.println(string);
                    if ("byebye".equals(string)) {
                        System.out.println("退出！");
                        printStream.close();
                        scanner.close();
                        break;
                    }
                }
 
            }
 
            Socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
 
public class Main {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 6655);
        ExcuteServerInPut excuteServerInPut = new ExcuteServerInPut(socket);
        ExcuteServerOutPut excuteServerOutPut = new ExcuteServerOutPut(socket);
        new Thread(excuteServerInPut).start();
        new Thread(excuteServerOutPut).start();
        }
}
```



#  客户端输入 

```
package com.linjingc.top.imwebsocketdemo.socketService;

import java.io.IOException;
import java.io.PrintStream;
import java.net.Socket;
import java.util.Scanner;

class ExcuteServerInPut implements Runnable{//接收服务器的数据
    private Socket ToServer;
 
    ExcuteServerInPut(Socket ToServer){
        this.ToServer = ToServer;
    }
 
    @Override
    public void run() {
        try {
            Scanner scanner = new Scanner(ToServer.getInputStream());
               while (scanner.hasNext()){
                System.out.println(scanner.nextLine());
            }
            scanner.close();
            ToServer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
class ExcuteServerOutPut implements Runnable{//向服务器发送数据
 
    private Socket Socket;
    ExcuteServerOutPut(Socket Socket){
        this.Socket = Socket;
    }
 
    @Override
    public void run() {
        try {
            PrintStream printStream = new PrintStream(Socket.getOutputStream());
            Scanner scanner = new Scanner(System.in);
            scanner.useDelimiter("\n");
            System.out.println("*****************************************");
            System.out.println("***用户注册:useerName:同户名(仅限一次)***");
            System.out.println("***进入群聊:G:           退出群聊:exit***");
            System.out.println("***私聊:P-用户名         退出私聊:exit***");
            System.out.println("***********退出聊天室:byebye*************");
            while (true){
                if(scanner.hasNext()) {
                    String string = scanner.next();
                    printStream.println(string);
                    if ("byebye".equals(string)) {
                        System.out.println("退出！");
                        printStream.close();
                        scanner.close();
                        break;
                    }
                }
 
            }
 
            Socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
 
public class Main {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 6655);
        ExcuteServerInPut excuteServerInPut = new ExcuteServerInPut(socket);
        ExcuteServerOutPut excuteServerOutPut = new ExcuteServerOutPut(socket);
        new Thread(excuteServerInPut).start();
        new Thread(excuteServerOutPut).start();
        }
}
```





# 客户端输出

```
package com.linjingc.top.imwebsocketdemo.socketService;

import java.io.IOException;
import java.io.PrintStream;
import java.net.Socket;
import java.util.Scanner;

class ExcuteServerInPut implements Runnable{//接收服务器的数据
    private Socket ToServer;
 
    ExcuteServerInPut(Socket ToServer){
        this.ToServer = ToServer;
    }
 
    @Override
    public void run() {
        try {
            Scanner scanner = new Scanner(ToServer.getInputStream());
               while (scanner.hasNext()){
                System.out.println(scanner.nextLine());
            }
            scanner.close();
            ToServer.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
class ExcuteServerOutPut implements Runnable{//向服务器发送数据
 
    private Socket Socket;
    ExcuteServerOutPut(Socket Socket){
        this.Socket = Socket;
    }
 
    @Override
    public void run() {
        try {
            PrintStream printStream = new PrintStream(Socket.getOutputStream());
            Scanner scanner = new Scanner(System.in);
            scanner.useDelimiter("\n");
            System.out.println("*****************************************");
            System.out.println("***用户注册:useerName:同户名(仅限一次)***");
            System.out.println("***进入群聊:G:           退出群聊:exit***");
            System.out.println("***私聊:P-用户名         退出私聊:exit***");
            System.out.println("***********退出聊天室:byebye*************");
            while (true){
                if(scanner.hasNext()) {
                    String string = scanner.next();
                    printStream.println(string);
                    if ("byebye".equals(string)) {
                        System.out.println("退出！");
                        printStream.close();
                        scanner.close();
                        break;
                    }
                }
 
            }
 
            Socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
 
 
public class Main {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 6655);
        ExcuteServerInPut excuteServerInPut = new ExcuteServerInPut(socket);
        ExcuteServerOutPut excuteServerOutPut = new ExcuteServerOutPut(socket);
        new Thread(excuteServerInPut).start();
        new Thread(excuteServerOutPut).start();
        }
}
```

