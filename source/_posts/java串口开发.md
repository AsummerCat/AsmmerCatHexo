---
title: java串口开发
date: 2023-02-11 02:04:59
tags: [java,串口]

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
rxtxParallel.dllrxtxSerial.dll
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
        @SuppressWarnings("unchecked")
        Enumeration<CommPortIdentifier> portList = CommPortIdentifier.getPortIdentifiers();
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
            serialPort.setSerialPortParams(
                    baudrate, SerialPort.DATABITS_8,
                    SerialPort.STOPBITS_1,
                    SerialPort.PARITY_NONE);
//            log.info("打开串口成功port:{}", portName);
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
    public static void closePort(
            SerialPort serialPort) {
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
package com.linjingc.guidemo.windows;

import com.linjingc.guidemo.comm.SerialComKit;
import gnu.io.SerialPort;
import gnu.io.SerialPortEvent;
import gnu.io.SerialPortEventListener;
import lombok.extern.slf4j.Slf4j;

import java.awt.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

@Slf4j
public class CommListenner implements SerialPortEventListener {
    /**
     * 类型.具体是下发了什么控制
     */
    public static int type = 0;

    public Map<String, SerialPort> map = new HashMap();

    public static String[] listData;
    public static String mPort;
    public static SerialPort serialPort;

    public static ArrayList<Map> objects = new ArrayList<>();

    public static String BleMac = null;
    public static String NBValue = null;
    public static String Version = null;

    @Override
    public void serialEvent(SerialPortEvent serialPortEvent) {
        switch (serialPortEvent.getEventType()) {
            //串口存在有效数据
            case SerialPortEvent.DATA_AVAILABLE:
                byte[] bytes = SerialComKit.readFromPort(JFrameWindowns.serialPort);
                String str = new String(bytes);
                int index = str.indexOf("+");
                str = str.substring(index + 1);

                //检测模块
                check(str);
                if (type == 2) {
                    printResults(str);
                    JFrameWindowns.resVersionText.setText(str);
                    //模块版本检测
                    JFrameWindowns.label2.setText("          PASS");
                    JFrameWindowns.label2.setForeground(Color.green);
                    Map map2 = new HashMap();
                    map2.put("Testcode", "GN02");
                    map2.put("Testresult", "PASS");
                    map2.put("Testvalue", str);
                    objects.add(map2);
                } else if (type == 15) {
                    //过站信息检测
                }
                break;
            // 2.输出缓冲区已清空
            case SerialPortEvent.OUTPUT_BUFFER_EMPTY:
//                log.error("输出缓冲区已清空");
                break;
            // 3.清除待发送数据
            case SerialPortEvent.CTS:
//                log.error("清除待发送数据");
                break;
            // 4.待发送数据准备好了
            case SerialPortEvent.DSR:
//                log.error("待发送数据准备好了");
                break;
            // 10.通讯中断
            case SerialPortEvent.BI:
//                log.error("与串口设备通讯中断");
                break;
            default:
                break;
        }
    }


    /**
     * 总校验调用
     *
     * @param str
     */
    public void check(String str) {
        //按键测试(复位键)
        boolean pc = powerCheck(str);
        if (pc) {
            return;
        }

        switch (type) {
            case 1:
                //进入测试模式
                versionCheck(str);
                break;
            case 2:
                break;
            case 3:
                //FLASH测试
                flashCheck(str);
                break;
            case 4:
                //3轴加速度计测试
                threeAxisSpeedCheck(str);
                break;
            case 5:
                //电池电量检测
                batteryCheck(str);
                break;
            case 6:
                //灯板光感测试 无需监听返回
                break;
            case 7:
                //灯板光感器件测试
                lampPanelPhotosensitiveDeviceCheck(str);
                break;
            case 8:
                //按键测试(电源) 无需监听返回
                break;
            case 9:
                //NB网络测试
                nbNetworkCheck(str);
                break;
            case 10:
                //蓝牙测试
                bluetoothCheck(str);
                break;
            case 11:
                //红外不带盔测试
                infraRedOutWearCheck(str);
                break;
            case 14:
                //红外带盔测试
                infraRedInWearCheck(str);
                break;
            case 12:
                //  进入仓储模式
                enterStorageModeCheck(str);
                break;

            default:

        }


    }


    /**
     * 进入测试模式
     *
     * @param str 返回数据集
     */
    public void versionCheck(String str) {
        printResults(str);
        //进入测试模式
        String[] args = str.split(":");
//        log.info("进入测试模式:" + args[1]);
        JFrameWindowns.resTestText.setText(str);
        if (args[1].trim().equals("ON")) {
            JFrameWindowns.label1.setText("          PASS");
            JFrameWindowns.label1.setForeground(Color.green);

        } else {
            JFrameWindowns.label1.setText("            NG");
            JFrameWindowns.label1.setForeground(Color.red);
        }
        Map map1 = new HashMap();
        map1.put("Testcode", "GN01");
        if (args[1].trim().equals("ON")) {
            map1.put("Testresult", "PASS");
        } else {
            map1.put("Testresult", "NG");
        }
        map1.put("Testvalue", "");
        objects.add(map1);
    }

    /**
     * FLASH测试
     *
     * @param str
     */
    public void flashCheck(String str) {
        printResults(str);
        String[] args = str.split(":");
        JFrameWindowns.resFlashText.setText(str);
//        log.info("FLASH测试结果:" + args[1]);
        if (args[1].trim().equals("OK")) {
            JFrameWindowns.label3.setText("          PASS");
            JFrameWindowns.label3.setForeground(Color.green);
        } else {
            JFrameWindowns.label3.setText("            NG");
            JFrameWindowns.label3.setForeground(Color.red);
        }

        Map map3 = new HashMap();
        map3.put("Testcode", "GN03");
        map3.put("Testresult", "PASS");
        map3.put("Testvalue", "");
        objects.add(map3);
    }

    /**
     * 3轴加速度计测试
     */
    public void threeAxisSpeedCheck(String str) {
        printResults(str);
        //3轴加速度计测试
        String value = str.substring(5);
        JFrameWindowns.resThreeAxisSpeedText.setText(str);
//        log.info("3轴加速度计测试:" + value);
        String[] args = value.split(",");
        String x = args[0].substring(2);
        String y = args[1].substring(2);
        String z = args[2].substring(2);
//        log.info("3轴加速度计测试x:" + Integer.parseInt(x));
//        log.info("3轴加速度计测试y:" + Integer.parseInt(y));
//        log.info("3轴加速度计测试z:" + Integer.parseInt(z.trim()));
        int a = Integer.parseInt(x.trim()) + Integer.parseInt(y.trim()) + Integer.parseInt(z.trim());
//        log.info("3轴加速度计测试:" + a);
        if (a > 950) {
            JFrameWindowns.label4.setText("          PASS");
            JFrameWindowns.label4.setForeground(Color.green);
        } else {
            JFrameWindowns.label4.setText("            NG");
            JFrameWindowns.label4.setForeground(Color.red);

        }
        Map map4 = new HashMap();
        map4.put("Testcode", "GN04");
        if (a > 950) {
            map4.put("Testresult", "PASS");
        } else {
            map4.put("Testresult", "NG");
        }
        map4.put("Testvalue", "");
        objects.add(map4);
    }

    /**
     * 电池电量检测
     */
    public void batteryCheck(String str) {
        printResults(str);
        //电池电量检测
        JFrameWindowns.resBatteryText.setText(str);
        String value = str.substring(4);
//        log.info("电池电量检测测试:" + Integer.parseInt(value.trim()));

        if (Integer.parseInt(value.trim()) > 3750) {
            JFrameWindowns.label5.setText("          PASS");
            JFrameWindowns.label5.setForeground(Color.green);
        } else {
            JFrameWindowns.label5.setText("            NG");
            JFrameWindowns.label5.setForeground(Color.red);
        }
        Map map5 = new HashMap();
        map5.put("Testcode", "GN05");
        if (Integer.parseInt(value.trim()) > 3750) {
            map5.put("Testresult", "PASS");
        } else {
            map5.put("Testresult", "NG");
        }
        map5.put("Testvalue", "");
        objects.add(map5);
    }


    /**
     * 灯板光感器件测试
     *
     * @param str
     */
    public void lampPanelPhotosensitiveDeviceCheck(String str) {
        printResults(str);
        //灯板光感器件测试
        String value = str.substring(6);
//        log.info("灯板光感器件测试:" + Integer.parseInt(value.trim()));
        JFrameWindowns.resLampPomeText.setText(str);
        if (Integer.parseInt(value.trim()) > 3801) {
            JFrameWindowns.label7.setText("          PASS");
            JFrameWindowns.label7.setForeground(Color.green);
        } else {
            JFrameWindowns.label7.setText("            NG");
            JFrameWindowns.label7.setForeground(Color.red);
        }


        Map map7 = new HashMap();
        map7.put("Testcode", "GN07");
        if (Integer.parseInt(value.trim()) > 3801) {
            map7.put("Testresult", "PASS");
        } else {
            map7.put("Testresult", "NG");
        }
        map7.put("Testvalue", "");
        objects.add(map7);
    }


    /**
     * NB网络测试
     *
     * @param str
     */
    public void nbNetworkCheck(String str) {
        printResults(str);

        //NB网络测试
        JFrameWindowns.resNbNetworkText.setText(str);
        JFrameWindowns.label9.setText("          PASS");
        JFrameWindowns.label9.setForeground(Color.green);
        Map map9 = new HashMap();
        map9.put("Testcode", "GN09");
        map9.put("Testresult", "PASS");
        map9.put("Testvalue", str);
        objects.add(map9);
    }

    /**
     * 蓝牙测试
     *
     * @param str
     */
    public void bluetoothCheck(String str) {
        printResults(str);

        //蓝牙
        JFrameWindowns.resBluetoothText.setText(str);
        JFrameWindowns.label10.setText("          PASS");
        JFrameWindowns.label10.setForeground(Color.green);
        //label102.setText(str);
        Map map10 = new HashMap();
        map10.put("Testcode", "GN010");
        map10.put("Testresult", "PASS");
        map10.put("Testvalue", str);
        objects.add(map10);
    }


    /**
     * 按键测试(复位键)
     *
     * @param str
     */
    public boolean powerCheck(String str) {
        if ("KEY:POWER".equals(str)) {
            JFrameWindowns.resKeyText.setText(str);
            printResults(str);
            return true;
        }
        return false;
    }

    /**
     * 红外未带盔测试
     *
     * @param str
     */
    public void infraRedOutWearCheck(String str) {
        printResults(str);

        //红外未带盔测试
        JFrameWindowns.resInfraRedOutWearText.setText(str);
        JFrameWindowns.label11.setText("          PASS");
        JFrameWindowns.label11.setForeground(Color.green);
        Map map11 = new HashMap();
        map11.put("Testcode", "GN11");
        map11.put("Testresult", "PASS");
        map11.put("Testvalue", "");
        objects.add(map11);
    }

    /**
     * 红外带盔测试
     *
     * @param str
     */
    public void infraRedInWearCheck(String str) {
        printResults(str);

        JFrameWindowns.resInfraRedInWearText.setText(str);
        //红外带头盔
        if (str.contains("ps")) {
            int a = str.indexOf("ps");
            int b = Integer.parseInt(str.substring(a + 2).trim());
            if (b > 2500 || b < 4905) {
                JFrameWindowns.label12.setText("            PASS");
                JFrameWindowns.label12.setForeground(Color.green);
            } else {
                JFrameWindowns.label12.setText("            NG");
                JFrameWindowns.label12.setForeground(Color.red);
            }
        } else {
            JFrameWindowns.label12.setText("            NG");
            JFrameWindowns.label12.setForeground(Color.red);
        }

        Map map14 = new HashMap();
        map14.put("Testcode", "GD01");
        map14.put("Testresult", "PASS");
        map14.put("Testvalue", "");
        objects.add(map14);
    }

    /**
     * 进入仓储模式
     */
    public void enterStorageModeCheck(String str) {
        printResults(str);
        //进入仓储模式
        JFrameWindowns.resEnterStorageModeText.setText(str);
        if (str.substring(0, 5).equals("MODE:")) {
            String[] args = str.split(":");
//            log.info("进入仓储模式:" + args[1]);
            if (args[1].trim().equals("OK")) {
                JFrameWindowns.label13.setText("          PASS");
                JFrameWindowns.label13.setForeground(Color.green);
            } else {
                JFrameWindowns.label13.setText("            NG");
                JFrameWindowns.label13.setForeground(Color.red);
            }

            Map map12 = new HashMap();
            map12.put("Testcode", "GN012");
            if (args[1].trim().equals("OK")) {
                map12.put("Testresult", "PASS");
            } else {
                map12.put("Testresult", "NG");
            }
            map12.put("Testvalue", "进入仓储模式");
            objects.add(map12);
        }
    }

    public static void printResults(String str){
        log.info("收到+:" + str);
        CommListenner.type = 999999;
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