---
title: 统计学-常用统计算法JAVA实现-工具类
date: 2019-04-10 21:02:59
tags: 统计学 
---

# 统计学-常用统计算法JAVA实现-工具类

[参考地址java实现](https://so.csdn.net/so/search/s.do?q=%E5%B8%B8%E7%94%A8%E7%BB%9F%E8%AE%A1%E7%AE%97%E6%B3%95JAVA%E5%AE%9E%E7%8E%B0&t=blog&u=china_melancholy)



<!--more-->

# 工具类 

```
package com.mdt.newdrugreview.utils;

import java.math.BigDecimal;

/**
 *  * @类描述：  一个工具类，为了保证计算准确性，将double之间的运算转换为BigDecimal之间的运算 <br/>
 *  * @项目名称：Statistics   <br/>
 *  * @包名： descrptive   <br/>
 *  * @类名称：Mutil   <br/>
 */
public class Mutil {
    /**
     *  * @描述: 加法 <br/>
     *  * @方法名: add <br/>
     */
    public static double add(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.add(b2).doubleValue();
    }

    /**
     *  * @描述: 减法 <br/>
     *  * @方法名: subtract <br/>
     */
    public static double subtract(double v1, double v2) {
        BigDecimal b1 = new BigDecimal(Double.toString(v1));
        BigDecimal b2 = new BigDecimal(Double.toString(v2));
        return b1.subtract(b2).doubleValue();
    }

    /**
     *  * @描述: 乘法 <br/>
     *  * @方法名: mul <br/>
     */
    public static double multiply(double d1, double d2) {// 进行乘法运算
        BigDecimal b1 = new BigDecimal(d1);
        BigDecimal b2 = new BigDecimal(d2);
        return b1.multiply(b2).doubleValue();
    }

    /**
     *  * @描述: 除法 ，四舍五入<br/>
     *  * @方法名: div <br/>
     */
    public static double divide(double d1, double d2, int len) {// 进行除法运算
        BigDecimal b1 = new BigDecimal(d1);
        BigDecimal b2 = new BigDecimal(d2);

        return b1.divide(b2, len, BigDecimal.ROUND_HALF_UP).doubleValue();
    }

    /**
     *  * @描述:  除法，四舍五入取整数 ,例如：5/2=3(2.5四舍五入); 5/3=2(1.6四舍五入);<br/>
     */
    public static double divide(double d1, double d2) {// 进行除法运算
        BigDecimal b1 = new BigDecimal(d1);
        BigDecimal b2 = new BigDecimal(d2);
        return b1.divide(b2, BigDecimal.ROUND_HALF_UP).doubleValue();
    }

    /**
     *  * @描述: 四舍五入 <br/>
     */
    public static double round(double d, int len) {
        BigDecimal b1 = new BigDecimal(d);
        BigDecimal b2 = new BigDecimal(1);
        // 任何一个数字除以1都是原数字
        // ROUND_HALF_UP是BigDecimal的一个常量，表示进行四舍五入的操作
        return b1.divide(b2, len, BigDecimal.ROUND_HALF_UP).doubleValue();
    }

    public static void main(String[] args) {
        double a = 10;
        double b = 3;
        System.out.println(divide(a, b, 2));
        System.out.println(divide(a, b));
    }
}

```