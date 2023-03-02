---
title: 串口Modbus协议开发
date: 2023-03-02 10:25:45
tags: [java,串口,Modbus协议]
---
# Modbus协议
主要用用于工业化的产品
RTU TCP
RTU UDP
RTU 串口
# jdk/bin也需要引入两个dll
```
rxtxParallel.dll    
rxtxSerial.dll
```
# 引入相关工具类jar
由于依赖不好下载 建议直接git源码编译 modbus4j
github地址: ` https://github.com/MangoAutomation/modbus4j.git`
```
 <dependency>
            <groupId>com.infiniteautomation</groupId>
            <artifactId>modbus4j</artifactId>
            <version>3.1.1-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.rxtx</groupId>
            <artifactId>rxtx</artifactId>
            <version>2.1.7</version>
        </dependency>
        <dependency>
            <groupId>org.scream3r</groupId>
            <artifactId>jssc</artifactId>
            <version>2.8.0</version>
        </dependency>
```
<!--more-->

# 工具类
## 主要调用工具类
```
package com.linjingc.guidemo.rtu;

import com.linjingc.guidemo.util.InitComUtil;
import com.serotonin.modbus4j.ModbusFactory;
import com.serotonin.modbus4j.ModbusMaster;
import com.serotonin.modbus4j.exception.ModbusInitException;
import com.serotonin.modbus4j.exception.ModbusTransportException;
import com.serotonin.modbus4j.msg.ReadHoldingRegistersRequest;
import com.serotonin.modbus4j.msg.ReadHoldingRegistersResponse;
import com.serotonin.modbus4j.sero.io.StreamUtils;
import gnu.io.SerialPort;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.collections4.MapUtils;
import org.springframework.stereotype.Component;

/**
 * @author chengy
 */
@Component
@Slf4j
public class ModbusUtil {

    private static ModbusFactory modbusFactory;

    static {
        if (modbusFactory == null) {
            modbusFactory = new ModbusFactory();
        }
    }

    /**
     * 读取Mobus协议的RTU
     *
     * @param portName 串口名
     * @param baudRate 波特率
     * @param dataBits 数据位
     * @param stopBits 中止位
     * @param parity   校验位
     * @param slaveId  从机地址
     * @param offset   寄存器读取开始地址
     * @param quantity 读取的寄存器数量
     * @return
     */
    public static void getValueByRTU(int slaveId, int offset, int quantity) {
        SerialPortWrapperImpl com4 = new SerialPortWrapperImpl("COM4", 9600, SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE, 0, 0);
        ModbusMaster master = modbusFactory.createRtuMaster(com4);
        try {
            //设置超时时间
            master.setTimeout(1000);
            //设置重连次数
            master.setRetries(3);
            //初始化
            master.init();

            try {
                ReadHoldingRegistersRequest request = null;
                request = new ReadHoldingRegistersRequest(slaveId, offset, quantity);
                ReadHoldingRegistersResponse response = (ReadHoldingRegistersResponse) master.send(request);
                if (response.isException()) {
//                    log.info("读取电压设备数据失败," + response.getExceptionMessage());
                } else {
                    //16进制转换
                    String resData = StreamUtils.dumpHex(response.getData());
                    String resDataStr = "0x" + resData;
                    Integer decode = Integer.decode(resDataStr);
                }
            } catch (ModbusTransportException e) {
                throw new RuntimeException("读取电压设备数据失败");
            }

        } catch (ModbusInitException e) {
            throw new RuntimeException("读取电压设备连接失败");
        }
        //执行完毕销毁
        master.destroy();
    }


    /**
     * 读取电压 寄存器40009的数据
     *
     * @param comName  串口号 COM4
     * @param slaveId  1
     * @param offset   8 表示从4000开始第9个
     * @param quantity 1
     */
    public static Integer getValueByRTUByVoltage(int slaveId, int offset, int quantity) throws Exception {
        SerialPortWrapperImpl com = new SerialPortWrapperImpl(MapUtils.getString(InitComUtil.FileData, "电压设备COM", ""), 9600, SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE, 0, 0);
        ModbusMaster master = modbusFactory.createRtuMaster(com);
        //获取的电压值
        Integer resValue = 0;
        try {
            //设置超时时间
            master.setTimeout(5000);
            //设置重连次数
            master.setRetries(3);
            //初始化
            master.init();
            for (int i = 1; i <= 5; i++) {
                Integer integer = doNextValueByRTUByVoltage(master, slaveId, offset, quantity);
                Thread.sleep(500);
                resValue = resValue + integer;
            }
        } catch (ModbusInitException e) {
            throw new ModbusInitException("读取电压设备连接失败");
        }finally {
            //执行完毕销毁
            master.destroy();
            Thread.sleep(500);
        }
        return resValue;
    }

    private static Integer doNextValueByRTUByVoltage(ModbusMaster master, int slaveId, int offset, int quantity) {
        try {
            ReadHoldingRegistersRequest request = new ReadHoldingRegistersRequest(slaveId, offset, quantity);
            ReadHoldingRegistersResponse response = (ReadHoldingRegistersResponse) master.send(request);
            if (response.isException()) {
                log.error("读取电压设备数据失败," + response.getExceptionMessage());
            } else {
                //16进制转换
                String resData = StreamUtils.dumpHex(response.getData());
                String resDataStr = "0x" + resData;
                Integer resValue = Integer.decode(resDataStr);
                log.warn("输出寄存器电压:{}", resValue);
                return resValue;
            }
        } catch (ModbusTransportException e) {
            throw new RuntimeException("读取电压设备数据失败");
        }
        return 0;
    }

    /**
     * 读取电流 0x1001 单元数据
     *
     * @param slaveId  1
     * @param offset   0
     * @param quantity 1
     */
    public static Integer getValueByRTUByCurrent(int slaveId, int offset, int quantity) throws Exception {
        SerialPortWrapperImpl com = new SerialPortWrapperImpl(MapUtils.getString(InitComUtil.FileData, "电流设备COM", ""), 115200, SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE, 0, 0);
        ModbusMaster master = modbusFactory.createRtuMaster(com);
        //获取的电压值
        Integer resValue = 0;
        try {
            //设置超时时间
            master.setTimeout(5000);
            //设置重连次数
            master.setRetries(3);
            //初始化
            master.init();

            for (int i = 1; i <= 5; i++) {
                Integer integer = doNextValueByRTUByCurrent(master, slaveId, offset, quantity);
                resValue = resValue + integer;
            }
        } catch (ModbusInitException e) {
            throw new ModbusInitException("读取电流设备连接失败");
        }finally {
            //执行完毕销毁
            master.destroy();
            Thread.sleep(500);
        }

        return resValue;
    }

    private static Integer doNextValueByRTUByCurrent(ModbusMaster master, int slaveId, int offset, int quantity) {
        try {
            ReadHoldingRegistersRequest request = new ReadHoldingRegistersRequest(slaveId, offset, quantity);
            ReadHoldingRegistersResponse response = (ReadHoldingRegistersResponse) master.send(request);
            if (response.isException()) {
                log.error("读取电压设备数据失败," + response.getExceptionMessage());
            } else {
                //16进制转换
                String resData = StreamUtils.dumpHex(response.getData());
                String resDataStr = "0x" + resData;
                Integer resValue = Integer.decode(resDataStr);
                log.warn("读取模拟电源值:{}", resValue);
                return resValue;

            }
        } catch (ModbusTransportException e) {
            throw new RuntimeException("读取电压设备数据失败");
        }
        return 0;
    }


    public static void main(String[] args) throws Exception {
        ModbusUtil.getValueByRTUByCurrent(1, 4097, 1);
//        ModbusUtil.getValueByRTUByVoltage("com",1,8,1);

//        log.info("输出电流，休眠模式用 读取0x1001");
        try {
            //从0x1001开始
            ModbusUtil.getValueByRTUByCurrent(1, 4097, 1);
        } catch (Exception e) {
            e.printStackTrace();
        }
//        log.info("输出实时电流，休眠模式用 读取0x1003");
        try {
            ModbusUtil.getValueByRTUByCurrent(1, 4099, 1);
        } catch (Exception e) {
            e.printStackTrace();
        }

//        log.info("输入实时电流，充电模式 读取0x1005");
        try {
            //从0x1000开始 -0x10005
            ModbusUtil.getValueByRTUByCurrent(1, 4101, 1);
        } catch (Exception e) {
            e.printStackTrace();
        }
//        log.info("输入电流，充电模式 读取0x1006");
        try {
            //从0x1001开始
            ModbusUtil.getValueByRTUByCurrent(1, 4102, 1);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
## Modbus基础的一些请求类
```
package com.linjingc.guidemo.rtu;

import com.serotonin.modbus4j.ModbusFactory;
import com.serotonin.modbus4j.ModbusMaster;
import com.serotonin.modbus4j.exception.ErrorResponseException;
import com.serotonin.modbus4j.exception.ModbusInitException;
import com.serotonin.modbus4j.exception.ModbusTransportException;
import com.serotonin.modbus4j.ip.IpParameters;
import com.serotonin.modbus4j.locator.BaseLocator;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Modbus基础的一些请求类
 */
public class ModbusUtils {
    private static Logger log = LoggerFactory.getLogger(ModbusUtils.class);
    /**
     * 工厂。
     */
    static ModbusFactory modbusFactory;

    static {
        if (modbusFactory == null) {
            modbusFactory = new ModbusFactory();
        }
    }

    /**
     * 获取master
     *
     * @return
     * @throws ModbusInitException
     */
    public static ModbusMaster getMaster(String host, int port) throws ModbusInitException {
        IpParameters params = new IpParameters();
        params.setHost(host);
        params.setPort(port);
        //
        // modbusFactory.createRtuMaster(wapper); //RTU 协议
        // modbusFactory.createUdpMaster(params);//UDP 协议
        // modbusFactory.createAsciiMaster(wrapper);//ASCII 协议
        ModbusMaster master = modbusFactory.createTcpMaster(params, false);// TCP 协议
        master.init();

        return master;
    }

    public static ModbusMaster getRtuIpMaster(String host, int port) throws ModbusInitException {
        IpParameters params = new IpParameters();
        params.setHost(host);
        params.setPort(port);
        params.setEncapsulated(true);
        ModbusMaster master = modbusFactory.createTcpMaster(params, false);
        try {
            //设置超时时间
            master.setTimeout(1000);
            //设置重连次数
            master.setRetries(3);
            //初始化
            master.init();
        } catch (ModbusInitException e) {
            e.printStackTrace();
        }
        return master;
    }

    /**
     * @param portName 串口名
     * @param baudRate 波特率
     * @param dataBits 数据位
     * @param stopBits 中止位
     * @param parity   校验位
     * @return
     * @throws ModbusInitException
     */
    public static ModbusMaster getSerialPortRtuMaster(String portName, Integer baudRate, Integer dataBits,
                                                      Integer stopBits, Integer parity) {
        // 设置串口参数，串口是COM1，波特率是9600
//         SerialPortWrapperImpl wrapper = new SerialPortWrapperImpl("COM2", 9600,SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE, 0, 0);
        SerialPortWrapperImpl wrapper = new SerialPortWrapperImpl(portName, baudRate,
                dataBits, stopBits, parity, 0, 0);
        ModbusMaster master = modbusFactory.createRtuMaster(wrapper);
        try {
            //设置超时时间
            master.setTimeout(1000);
            //设置重连次数
            master.setRetries(3);
            //初始化
            master.init();
        } catch (ModbusInitException e) {
            log.error("串口连接异常~");
            e.printStackTrace();
        }
        return master;
    }

    /**
     * @param portName 串口名
     * @param baudRate 波特率
     * @param dataBits 数据位
     * @param stopBits 中止位
     * @param parity   校验位
     * @return
     * @throws ModbusInitException
     */
    public static ModbusMaster getSerialPortAsciiMaster(String portName, Integer baudRate, Integer dataBits,
                                                        Integer stopBits, Integer parity) {
        // 设置串口参数，串口是COM1，波特率是9600
        // SerialPortWrapperImpl wrapper = new SerialPortWrapperImpl("COM2", 9600,SerialPort.DATABITS_8, SerialPort.STOPBITS_1, SerialPort.PARITY_NONE, 0, 0);
        SerialPortWrapperImpl wrapper = new SerialPortWrapperImpl(portName, baudRate,
                dataBits, stopBits, parity, 0, 0);
        ModbusMaster master = modbusFactory.createAsciiMaster(wrapper);
        try {
            //设置超时时间
            master.setTimeout(1000);
            //设置重连次数
            master.setRetries(3);
            //初始化
            master.init();
        } catch (ModbusInitException e) {
            log.error("串口连接异常~");
            e.printStackTrace();
        }
        return master;
    }


    /**
     * 读取[01 Coil Status 0x]类型 开关数据
     *
     * @param slaveId slaveId
     * @param offset  位置
     * @return 读取值
     * @throws ModbusTransportException 异常
     * @throws ErrorResponseException   异常
     * @throws ModbusInitException      异常
     */
    public static Boolean readCoilStatus(ModbusMaster master, int slaveId, int offset)
            throws ModbusTransportException, ErrorResponseException, ModbusInitException {
        // 01 Coil Status
        BaseLocator<Boolean> loc = BaseLocator.coilStatus(slaveId, offset);
        Boolean value = master.getValue(loc);
        return value;
    }

    /**
     * 读取[03 Holding Register类型 2x]模拟量数据
     *
     * @param slaveId  slave Id
     * @param offset   位置
     * @param dataType 数据类型,来自com.serotonin.modbus4j.code.DataType
     * @return
     * @throws ModbusTransportException 异常
     * @throws ErrorResponseException   异常
     * @throws ModbusInitException      异常
     */
    public static Number readHoldingRegister(ModbusMaster master, int slaveId, int offset, int dataType)
            throws ModbusTransportException, ErrorResponseException, ModbusInitException {
        // 03 Holding Register类型数据读取
        BaseLocator<Number> loc = BaseLocator.holdingRegister(slaveId, offset, dataType);
        Number value = master.getValue(loc);
        return value;
    }

    /**
     * 读取[04 Input Registers 3x]类型 模拟量数据
     *
     * @param slaveId  slaveId
     * @param offset   位置
     * @param dataType 数据类型,来自com.serotonin.modbus4j.code.DataType
     * @return 返回结果
     * @throws ModbusTransportException 异常
     * @throws ErrorResponseException   异常
     * @throws ModbusInitException      异常
     */
    public static Number readInputRegisters(ModbusMaster master, int slaveId, int offset, int dataType)
            throws ModbusTransportException, ErrorResponseException, ModbusInitException {
        // 04 Input Registers类型数据读取
        BaseLocator<Number> loc = BaseLocator.inputRegister(slaveId, offset, dataType);
        Number value = master.getValue(loc);
        return value;
    }
}
```

## 输入流 拷贝来自modbus4j
```
package com.linjingc.guidemo.rtu;

/*
 * To change this template, choose Tools | Templates
 * and open the template in the editor.
 */

import jssc.SerialPort;

import java.io.IOException;
import java.io.InputStream;

/**
 * Class that wraps a {@link SerialPort} to provide {@link InputStream}
 * functionality. This stream also provides support for performing blocking
 * reads with timeouts.
 * <br>
 * It is instantiated by passing the constructor a {@link SerialPort} instance.
 * Do not create multiple streams for the same serial port unless you implement
 * your own synchronization.
 *
 * @author Charles Hache <chalz@member.fsf.org>
 * <p>
 * Attribution: https://github.com/therealchalz/java-simple-serial-connector
 */
/**
 * 输入流 拷贝来自modbus4j
 */
public class SerialInputStream extends InputStream {

    private SerialPort serialPort;
    private int defaultTimeout = 0;

    /**
     * Instantiates a SerialInputStream for the given {@link SerialPort} Do not
     * create multiple streams for the same serial port unless you implement
     * your own synchronization.
     *
     * @param sp The serial port to stream.
     */
    public SerialInputStream(SerialPort sp) {
        serialPort = sp;
    }

    /**
     * Set the default timeout (ms) of this SerialInputStream. This affects
     * subsequent calls to {@link #read()}, {@link #(int[])}, and
     * {@link #(int[], int, int)} The default timeout can be 'unset'
     * by setting it to 0.
     *
     * @param time The timeout in milliseconds.
     */
    public void setTimeout(int time) {
        defaultTimeout = time;
    }

    /**
     * Reads the next byte from the port. If the timeout of this stream has been
     * set, then this method blocks until data is available or until the timeout
     * has been hit. If the timeout is not set or has been set to 0, then this
     * method blocks indefinitely.
     */
    @Override
    public int read() throws IOException {
        return read(defaultTimeout);
    }

    /**
     * The same contract as {@link #read()}, except overrides this stream's
     * default timeout with the given timeout in milliseconds.
     *
     * @param timeout The timeout in milliseconds.
     * @return The read byte.
     * @throws IOException On serial port error or timeout
     */
    public int read(int timeout) throws IOException {
        byte[] buf = new byte[1];
        try {
            if (timeout > 0) {
                buf = serialPort.readBytes(1, timeout);
            } else {
                buf = serialPort.readBytes(1);
            }
            return buf[0];
        } catch (Exception e) {
            throw new IOException(e);
        }
    }

    /**
     * Non-blocking read of up to buf.length bytes from the stream. This call
     * behaves as read(buf, 0, buf.length) would.
     *
     * @param buf The buffer to fill.
     * @return The number of bytes read, which can be 0.
     * @throws IOException on error.
     */
    @Override
    public int read(byte[] buf) throws IOException {
        return read(buf, 0, buf.length);
    }

    /**
     * Non-blocking read of up to length bytes from the stream. This method
     * returns what is immediately available in the input buffer.
     *
     * @param buf    The buffer to fill.
     * @param offset The offset into the buffer to start copying data.
     * @param length The maximum number of bytes to read.
     * @return The actual number of bytes read, which can be 0.
     * @throws IOException on error.
     */
    @Override
    public int read(byte[] buf, int offset, int length) throws IOException {

        if (buf.length < offset + length) {
            length = buf.length - offset;
        }

        int available = this.available();

        if (available > length) {
            available = length;
        }

        try {
            byte[] readBuf = serialPort.readBytes(available);
//            System.arraycopy(readBuf, 0, buf, offset, length);
            System.arraycopy(readBuf, 0, buf, offset, readBuf.length);
            return readBuf.length;
        } catch (Exception e) {
            throw new IOException(e);
        }
    }

    /**
     * Blocks until buf.length bytes are read, an error occurs, or the default
     * timeout is hit (if specified). This behaves as blockingRead(buf, 0,
     * buf.length) would.
     *
     * @param buf The buffer to fill with data.
     * @return The number of bytes read.
     * @throws IOException On error or timeout.
     */
    public int blockingRead(byte[] buf) throws IOException {
        return blockingRead(buf, 0, buf.length, defaultTimeout);
    }

    /**
     * The same contract as {@link #blockingRead(byte[])} except overrides this
     * stream's default timeout with the given one.
     *
     * @param buf     The buffer to fill.
     * @param timeout The timeout in milliseconds.
     * @return The number of bytes read.
     * @throws IOException On error or timeout.
     */
    public int blockingRead(byte[] buf, int timeout) throws IOException {
        return blockingRead(buf, 0, buf.length, timeout);
    }

    /**
     * Blocks until length bytes are read, an error occurs, or the default
     * timeout is hit (if specified). Saves the data into the given buffer at
     * the specified offset. If the stream's timeout is not set, behaves as
     * {@link #read(byte[], int, int)} would.
     *
     * @param buf    The buffer to fill.
     * @param offset The offset in buffer to save the data.
     * @param length The number of bytes to read.
     * @return the number of bytes read.
     * @throws IOException on error or timeout.
     */
    public int blockingRead(byte[] buf, int offset, int length) throws IOException {
        return blockingRead(buf, offset, length, defaultTimeout);
    }

    /**
     * The same contract as {@link #blockingRead(byte[], int, int)} except
     * overrides this stream's default timeout with the given one.
     *
     * @param buf     The buffer to fill.
     * @param offset  Offset in the buffer to start saving data.
     * @param length  The number of bytes to read.
     * @param timeout The timeout in milliseconds.
     * @return The number of bytes read.
     * @throws IOException On error or timeout.
     */
    public int blockingRead(byte[] buf, int offset, int length, int timeout) throws IOException {
        if (buf.length < offset + length) {
            throw new IOException("Not enough buffer space for serial data");
        }

        if (timeout < 1) {
            return read(buf, offset, length);
        }

        try {
            byte[] readBuf = serialPort.readBytes(length, timeout);
            System.arraycopy(readBuf, 0, buf, offset, length);
            return readBuf.length;
        } catch (Exception e) {
            throw new IOException(e);
        }
    }

    @Override
    public int available() throws IOException {
        int ret;
        try {
            ret = serialPort.getInputBufferBytesCount();
            if (ret >= 0) {
                return ret;
            }
            throw new IOException("Error checking available bytes from the serial port.");
        } catch (Exception e) {
            throw new IOException("Error checking available bytes from the serial port.");
        }
    }

}
 
```

## 输出流 拷贝来自modbus4j
```
package com.linjingc.guidemo.rtu;

/**
 * Copyright (c) 2009-2020 Freedomotic Team http://www.freedomotic-iot.com
 * <p>
 * This file is part of Freedomotic
 * <p>
 * This Program is free software; you can redistribute it and/or modify it under
 * the terms of the GNU General Public License as published by the Free Software
 * Foundation; either version 2, or (at your option) any later version.
 * <p>
 * This Program is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
 * FOR A PARTICULAR PURPOSE. See the GNU General Public License for more
 * details.
 * <p>
 * You should have received a copy of the GNU General Public License along with
 * Freedomotic; see the file COPYING. If not, see
 * <http://www.gnu.org/licenses/>.
 */

import jssc.SerialPort;
import jssc.SerialPortException;

import java.io.IOException;
import java.io.OutputStream;

/**
 * Class that wraps a {@link SerialPort} to provide {@link OutputStream}
 * functionality.
 * <br>
 * It is instantiated by passing the constructor a {@link SerialPort} instance.
 * Do not create multiple streams for the same serial port unless you implement
 * your own synchronization.
 *
 * @author Charles Hache <chalz@member.fsf.org>
 *
 * Attribution: https://github.com/therealchalz/java-simple-serial-connector
 *
 */

/**
 * 输出流 拷贝来自modbus4j
 */
public class SerialOutputStream extends OutputStream {

    SerialPort serialPort;

    /**
     * Instantiates a SerialOutputStream for the given {@link SerialPort} Do not
     * create multiple streams for the same serial port unless you implement
     * your own synchronization.
     *
     * @param sp The serial port to stream.
     */
    public SerialOutputStream(SerialPort sp) {
        serialPort = sp;
    }

    @Override
    public void write(int b) throws IOException {
        try {
            serialPort.writeInt(b);
        } catch (SerialPortException e) {
            throw new IOException(e);
        }
    }

    @Override
    public void write(byte[] b) throws IOException {
        write(b, 0, b.length);

    }

    @Override
    public void write(byte[] b, int off, int len) throws IOException {
        byte[] buffer = new byte[len];
        System.arraycopy(b, off, buffer, 0, len);
        try {
            serialPort.writeBytes(buffer);
        } catch (SerialPortException e) {
            throw new IOException(e);
        }
    }
}
 
 
```

## 自定义消息构建器
```
package com.linjingc.guidemo.rtu;

/**
 * Copyright (c) 2009-2020 Freedomotic Team http://www.freedomotic-iot.com
 * <p>
 * This file is part of Freedomotic
 * <p>
 * This Program is free software; you can redistribute it and/or modify it under
 * the terms of the GNU General Public License as published by the Free Software
 * Foundation; either version 2, or (at your option) any later version.
 * <p>
 * This Program is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
 * FOR A PARTICULAR PURPOSE. See the GNU General Public License for more
 * details.
 * <p>
 * You should have received a copy of the GNU General Public License along with
 * Freedomotic; see the file COPYING. If not, see
 * <http://www.gnu.org/licenses/>.
 */

import com.serotonin.modbus4j.serial.SerialPortWrapper;
import jssc.SerialPort;
import jssc.SerialPortException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.InputStream;
import java.io.OutputStream;

/**
 * 自定义消息构建器
 */
public class SerialPortWrapperImpl implements SerialPortWrapper {

    private static final Logger LOG = LoggerFactory.getLogger(SerialPortWrapperImpl.class);
    private SerialPort port;
    private String commPortId;
    private int baudRate;
    private int dataBits;
    private int stopBits;
    private int parity;
    private int flowControlIn;
    private int flowControlOut;

    public SerialPortWrapperImpl(String commPortId, int baudRate, int dataBits, int stopBits, int parity, int flowControlIn,
                                 int flowControlOut) {

        this.commPortId = commPortId;
        this.baudRate = baudRate;
        this.dataBits = dataBits;
        this.stopBits = stopBits;
        this.parity = parity;
        this.flowControlIn = flowControlIn;
        this.flowControlOut = flowControlOut;

        port = new SerialPort(this.commPortId);

    }

    @Override
    public void close() throws Exception {
        port.closePort();
        //listeners.forEach(PortConnectionListener::closed);
        LOG.debug("Serial port {} closed", port.getPortName());
    }

    @Override
    public void open() {
        try {
            port.openPort();
            port.setParams(this.getBaudRate(), this.getDataBits(), this.getStopBits(), this.getParity());
            port.setFlowControlMode(this.getFlowControlIn() | this.getFlowControlOut());

            //listeners.forEach(PortConnectionListener::opened);
            LOG.debug("Serial port {} opened", port.getPortName());
        } catch (SerialPortException ex) {
            LOG.error("Error opening port : {} for {} ", port.getPortName(), ex);
        }
    }

    @Override
    public InputStream getInputStream() {
        return new SerialInputStream(port);
    }

    @Override
    public OutputStream getOutputStream() {
        return new SerialOutputStream(port);
    }

    @Override
    public int getBaudRate() {
        return baudRate;
        //return SerialPort.BAUDRATE_9600;
    }

    public int getFlowControlIn() {
        return flowControlIn;
        //return SerialPort.FLOWCONTROL_NONE;
    }

    public int getFlowControlOut() {
        return flowControlOut;
        //return SerialPort.FLOWCONTROL_NONE;
    }

    @Override
    public int getDataBits() {
        return dataBits;
        //return SerialPort.DATABITS_8;
    }

    @Override
    public int getStopBits() {
        return stopBits;
        //return SerialPort.STOPBITS_1;
    }

    @Override
    public int getParity() {
        return parity;
        //return SerialPort.PARITY_NONE;
    }
}
 
```