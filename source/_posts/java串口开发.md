---
title: java串口开发 
date: 2023-02-11 02:04:59 
tags: [java,串口]
---

# 首先需要引入对应的依赖

```
        <dependency>
            <groupId>org.bidib.jbidib.org.qbang.rxtx</groupId>
            <artifactId>rxtxcomm</artifactId>
            <version>2.2</version>
        </dependency>
```

## 需要在jdk环境下的 jdk/bin目录中添加

```
rxtxParallel.dll    
rxtxSerial.dll
```

<!---more-->

## 创建串口工具类

```
package com.linjingc.guidemo.comm;

import gnu.io.*;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.*;

/**
 * 串口通信工具
 */
@Slf4j
public class SerialComKit {
    public static Map<String, SerialPort> map = new HashMap();
    public static List<String> portNameList = new ArrayList<>(9);

    /**
     * 查询可用串口
     *
     * @return
     */
    public static List<String> findPort() {
        portNameList.clear();
        //获得当前所有可用串口
        @SuppressWarnings("unchecked") Enumeration<CommPortIdentifier> portList = CommPortIdentifier.getPortIdentifiers();
        //将可用串口名添加到List并返回该List
        while (portList.hasMoreElements()) {
            String portName = portList.nextElement().getName();
            portNameList.add(portName);
        }
        return portNameList;
    }

    /**
     * 开启串口
     *
     * @param portName 串口名称
     * @param baudrate 波特率
     * @return
     * @throws NoSuchPortException
     * @throws PortInUseException
     * @throws UnsupportedCommOperationException
     * @throws Exception
     */
    public static SerialPort openPort(String portName, int baudrate) throws NoSuchPortException, PortInUseException, UnsupportedCommOperationException {
        //通过端口名识别端口
        CommPortIdentifier portIdentifier = CommPortIdentifier.getPortIdentifier(portName);

        //打开端口，并给端口名字和一个timeout（打开操作的超时时间）
        CommPort commPort = portIdentifier.open(portName, 3000);

        //判断是不是串口
        if (commPort instanceof SerialPort) {
            SerialPort serialPort = (SerialPort) commPort;
            //设置串口的波特率等参数
            serialPort.setSerialPortParams(baudrate, SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE);
            return serialPort;
        } else {
            return null;
        }
    }

    /**
     * 关闭串口
     *
     * @param serialPort
     */
    public static void closePort(SerialPort serialPort) {
        if (serialPort != null) {
            try {
                serialPort.removeEventListener();
                serialPort.close();
                serialPort = null;
            } catch (Exception e) {
            }
        }
    }

    /**
     * 发送数据到串口
     *
     * @param serialPort
     * @param data
     * @return
     * @throws Exception
     */
    public static boolean sendToPort(SerialPort serialPort, byte[] data) throws Exception {
        OutputStream out = null;
        try {
            out = serialPort.getOutputStream();
            out.write(data);
            out.flush();
            return true;
        } catch (IOException e) {
            return false;
        } finally {
            try {
                if (out != null) {
                    out.close();
                    out = null;
                }
            } catch (IOException e) {
            }
        }

    }

    /**
     * 读取串口数据
     *
     * @param serialPort
     * @return 字节数组
     */
    public static byte[] readFromPort(SerialPort serialPort) {
        InputStream in = null;
        byte[] bytes = null;
        try {
            //需要延迟50毫秒 保证报文尽量完整
            Thread.sleep(50);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        try {
            in = serialPort.getInputStream();
            //获取buffer里的数据长度
            int bufflenth = in.available();
            while (bufflenth != 0) {
                //初始化byte数组为buffer中数据的长度
                bytes = new byte[bufflenth];
                in.read(bytes);
                bufflenth = in.available();
            }
        } catch (IOException e) {
            return null;
        } finally {
            try {
                if (in != null) {
                    in.close();
                    in = null;
                }
            } catch (IOException e) {
            }
        }
        return bytes;
    }

    /**
     * 添加监听事件
     *
     * @param port
     * @param listener
     * @throws TooManyListenersException
     */
    public static void addListener(SerialPort port, SerialPortEventListener listener) throws TooManyListenersException {
        //给串口添加监听器
        port.addEventListener(listener);
        //设置当有数据到达时唤醒监听接收线程
        port.notifyOnDataAvailable(true);
        //设置当通信中断时唤醒中断线程
        port.notifyOnBreakInterrupt(true);
    }


//    public static void main(String[] args) throws Exception {
//        //获取端口当前所有串口
//        //List<String> findPort = SerialComKit.findPort();
//        //System.out.println(findPort);
//
//        SerialPort serialPort = openPort("COM6", 115200);
//        map.put("COM6", serialPort);
//        Handle handle = new Handle();
//        handle.setPort(serialPort);
//        addListener(serialPort, handle);
//    }
}
```

## 串口监听器 实现SerialPortEventListener

```
package com.linjingc.guidemo.comm;

import gnu.io.SerialPort;
import gnu.io.SerialPortEvent;
import gnu.io.SerialPortEventListener;

public class Handle implements SerialPortEventListener {
    private SerialPort port;

    @Override
    public void serialEvent(SerialPortEvent event) {

        switch (event.getEventType()) {
            //串口存在有效数据
            case SerialPortEvent.DATA_AVAILABLE:
                byte[] bytes = SerialComKit.readFromPort(port);
                //String byteStr = new String(bytes, 0, bytes.length).trim();
                System.out.println("===========start===========");
                String str = new String(bytes);
                System.out.println(str);
                //System.out.println(new Date() + "【读到的字符串】：-----" + byteStr);
                //System.out.println(new Date() + "【字节数组转16进制字符串】：-----" + printHexString(bytes));
                System.out.println("===========end===========");
                break;
            // 2.输出缓冲区已清空
            case SerialPortEvent.OUTPUT_BUFFER_EMPTY:
                //log.error("输出缓冲区已清空");
                break;
            // 3.清除待发送数据
            case SerialPortEvent.CTS:
                //log.error("清除待发送数据");
                break;
            // 4.待发送数据准备好了
            case SerialPortEvent.DSR:
                //log.error("待发送数据准备好了");
                break;
            // 10.通讯中断
            case SerialPortEvent.BI:
                ///log.error("与串口设备通讯中断");
                break;
            default:
                break;
        }

    }

    public void setPort(SerialPort port) {
        this.port = port;
    }

}


```

## 使用

```
//打开串口
            SerialComKit.openPort(mPort.get(), 115200);
//创建监听端口         
            CommListenner handle = new CommListenner();
//打开串口监听            
            SerialComKit.addListener(serialPort, handle);
//发送请求            
            SerialComKit.sendToPort(serialPort, str.getBytes());

//            CommListenner.java监听器接收响应请求
```

## 问题1: 解决处理jar包直接启动(不走控制台启动)无法找到对应串口问题

```
/**
 * 处理jar包直接启动无法找到对应串口问题
 */
public class InitComLoadConfig {

    public static void initCom() {
        System.loadLibrary("win32com");
        String driverName = "com.sum.comm.Win32Driver";
        CommDriver driver;
        try {
            driver = (CommDriver) Class.forName(driverName).newInstance();
            driver.initialize();
        } catch (Exception ignored) {
        }
    }
}
```