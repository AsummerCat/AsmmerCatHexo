---
title: 统计学-T检验计算
date: 2019-04-10 21:15:24
tags: 统计学
---

# 统计学-T检验计算

## 导入pom文件

```
<!--科学计算类库-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>


```

<!--more-->

## 计算类

```
package com.mdt.newdrugreview.utils.tExamination;

import com.mdt.newdrugreview.utils.Mutil;
import org.apache.commons.math3.stat.StatUtils;

/**
 * 独立样本T检验
 *
 * @author cxc
 */
public class IndependentSamplesT {


    //test
    public static void main(String args[]) {
        double[] x1 = {7, 3, 3, 2, 3, 8, 8, 5, 8, 5, 5, 4, 6, 10, 10, 5, 1, 1, 4, 3, 5, 7, 1, 9, 2, 5, 2, 12, 15};
        double[] y1 = {5, 4, 4, 5, 5, 7, 8, 8, 9, 8, 3, 2, 5, 4, 4, 6, 7, 7, 5, 6, 4, 3, 2, 7, 6, 2, 8, 9, 7, 6};
        System.out.println("计算[独立样本T检测]:" + independentSamplesT(x1, y1));
    }


    /**
     *  * @描述: 独立样本T检验 ，用于检测群体x和群体y之间是否支持零假设<br/>
     *  * @方法名: independentSamplesT <br/>
     *  * @param x <br/>
     *  * @param y <br/>
     *  * @返回类型 double 分双重检测与单侧检测，结合自由度查询"临界值"对照表，如果实际值大于临界值
     * 表示不能接受零假设，否则零假设是最有力的假设 <br/>
     */
    public static double independentSamplesT(double[] x, double[] y) {
        double Xmean = StatUtils.mean(x);
        double Ymean = StatUtils.mean(y);
        int n1 = x.length;
        int n2 = y.length;
        double Xvariance = StatUtils.variance(x);
        double Yvariance = StatUtils.variance(y);
        return Mutil.divide(Mutil.subtract(Xmean, Ymean), Math.sqrt(Mutil.multiply(Mutil.divide(Mutil.add(Mutil.multiply((n1 - 1), Xvariance), Mutil.multiply((n2 - 1), Yvariance)), (n1 + n2 - 2), 2), Mutil.divide((n1 + n2), Mutil.multiply(n1, n2), 2))), 2);
    }
}
 
```