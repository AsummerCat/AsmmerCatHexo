---
title: 统计学-T检验计算
date: 2019-04-10 21:15:24
tags: [统计学]
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

# 计算类 

```
package com.mdt.newdrugreview.utils.tExamination;

import com.mdt.newdrugreview.utils.Mutil;
import org.apache.commons.math3.stat.StatUtils;

import java.math.BigDecimal;

/**
 * 独立样本T检验
 *
 * @author cxc
 */
public class IndependentSamplesT {

    //8, 8, 5, 8, 5, 5, 4, 6, 10, 10, 5, 1, 1, 4, 3, 5, 7, 1, 9, 2, 5, 2, 12, 15
//    , 7, 8, 8, 9, 8, 3, 2, 5, 4, 4, 6, 7, 7, 5, 6, 4, 3, 2, 7, 6, 2, 8, 9, 7, 6
    //test
    public static void main(String args[]) {
        double[] x1 = {7, 3, 3, 2, 3};
        double[] y1 = {5, 4, 4, 5, 5, 7};
        System.out.println("计算[独立样本T检测]:" + independentSamplesT(x1, y1));
    }


    /**
     *  * @描述: 独立样本T检验 ，用于检测群体x和群体y之间是否支持零假设<br/>
     *  * @方法名: independentSamplesT <br/>
     *  * @param x <br/>
     *  * @param y <br/>
     *  * @return <br/>
     *  * @返回类型 double 分双重检测与单侧检测，结合自由度查询"临界值"对照表，如果实际值大于临界值
     * 表示不能接受零假设，否则零假设是最有力的假设 <br/>
     *  * @创建人 micheal <br/>
     *  * @创建时间 2019年1月5日下午8:57:00 <br/>
     *  * @修改人 micheal <br/>
     *  * @修改时间 2019年1月5日下午8:57:00 <br/>
     *  * @修改备注 <br/>
     *  * @since <br/>
     *  * @throws  
     */
    public static double independentSamplesT(double[] x, double[] y) {
        double xMean = StatUtils.mean(x);
        double yMean = StatUtils.mean(y);
        int n1 = x.length;
        int n2 = y.length;
        double xVariance = StatUtils.variance(x);
        double yVariance = StatUtils.variance(y);
        System.out.println("自由度:" + (n1 + n2 - 2));
        //自由度
        int dof = getDOF(n1 + n2 - 2);
        //T值
        double tValue = Mutil.divide(Mutil.subtract(xMean, yMean), Math.sqrt(Mutil.multiply(Mutil.divide(Mutil.add(Mutil.multiply((n1 - 1), xVariance), Mutil.multiply((n2 - 1), yVariance)), (n1 + n2 - 2), 2), Mutil.divide((n1 + n2), Mutil.multiply(n1, n2), 5))), 3);
        System.out.println("t值:" + tValue);

        double pValue = resultPvalue(dof, tValue);
        System.out.println("p值:" + pValue);
        return pValue;
    }

    /**
     * 双侧
     * 获取T分布临界值获取 取近似自由度
     *
     * @param dof 自由度
     * @return
     */
    private static int getDOF(int dof) {
        //T分布临界值获取 取近似自由度
        if (dof > 40 && dof <= 45) {
            dof = 40;
        } else if (dof > 45 && dof <= 55) {
            dof = 50;
        } else if (dof > 55 && dof <= 65) {
            dof = 60;
        } else if (dof > 65 && dof <= 75) {
            dof = 70;
        } else if (dof > 75 && dof <= 85) {
            dof = 80;
        } else if (dof > 85 && dof < 95) {
            dof = 90;
        } else if (dof > 95 && dof <= 150) {
            dof = 100;
        } else if (dof > 150 && dof <= 350) {
            dof = 200;
        } else if (dof > 350 && dof <= 750) {
            dof = 500;
        } else if (dof > 750 && dof <= 1000) {
            dof = 1000;
        } else if (dof > 1000) {
            //超出最大
            dof = 10000;
        } else {
            return dof;
        }
        return dof;
    }


    /**
     * 根据T值临界表获取出 对比结果
     *
     * @return
     */
    private static double resultPvalue(int dof, double tValue) {
        TDistributionEnum tDistribution = TDistributionEnum.getValue(dof);
        if (tDistribution == null) {
            return 0.00;
        }
        BigDecimal tValueBigDecimal = new BigDecimal(Math.abs(tValue));
        //p值0.5判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotFive())) < 1) {
            return 0.5;
        }
        //p值0.001判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroOne())) > -1) {
            return 0.001;
        }


        //p值0.5-0.2判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotFive())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotTwo())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotTwo())) == 0) {
                return 0.2;
            } else {
                return 0.35;
            }
        }
        //p值0.2-0.1判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotTwo())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotOne())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotOne())) == 0) {
                return 0.1;
            } else {
                return 0.15;
            }
        }
        //p值0.1-0.05判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotOne())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroFive())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroFive())) == 0) {
                return 0.05;
            } else {
                return 0.075;
            }
        }
        //p值0.05-0.02判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroFive())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroTwo())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroTwo())) == 0) {
                return 0.02;
            } else {
                return 0.035;
            }
        }
        //p值0.02-0.01判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroTwo())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroOne())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroOne())) == 0) {
                return 0.01;
            } else {
                return 0.015;
            }
        }
        //p值0.01-0.005判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroOne())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroFive())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroFive())) == 0) {
                return 0.005;
            } else {
                return 0.0075;
            }
        }
        //p值0.005-0.002判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroFive())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroTwo())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroTwo())) == 0) {
                return 0.002;
            } else {
                return 0.0035;
            }
        }
        //p值0.002-0.001判断
        if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroTwo())) > 0 && tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroOne())) < 1) {
            if (tValueBigDecimal.compareTo(new BigDecimal(tDistribution.getZeroDotZeroZeroOne())) == 0) {
                return 0.001;
            } else {
                return 0.0015;
            }
        }
        return 0.00;
    }


}

```



# 枚举类 维护 T值临界表 双侧

```
package com.mdt.newdrugreview.utils.tExamination;

/**
 * T值临界表 双侧
 *
 * @author cxc
 * @date 2019年7月2日10:23:22
 */
public enum TDistributionEnum {

    ONE(1, 1, 3.078, 6.314, 12.706, 31.821, 63.657, 127.321, 318.309, 636.619),
    TWO(2, 0.816, 1.886, 2.92, 4.303, 6.965, 9.925, 14.089, 22.327, 31.599),
    THREE(3, 0.765, 1.638, 2.353, 3.182, 4.541, 5.841, 7.453, 10.215, 12.924),
    FOUR(4, 0.741, 1.533, 2.132, 2.776, 3.747, 4.604, 5.598, 7.173, 8.61),
    FIVE(5, 0.727, 1.476, 2.015, 2.571, 3.365, 4.032, 4.773, 5.893, 6.869),
    SIX(6, 0.718, 1.44, 1.943, 2.447, 3.143, 3.707, 4.317, 5.208, 5.959),
    SEVEN(7, 0.711, 1.415, 1.895, 2.365, 2.998, 3.499, 4.029, 4.785, 5.408),
    EIGHT(8, 0.706, 1.397, 1.86, 2.306, 2.896, 3.355, 3.833, 4.501, 5.041),
    NINE(9, 0.703, 1.383, 1.833, 2.262, 2.821, 3.25, 3.69, 4.297, 4.781),
    TEN(10, 0.7, 1.372, 1.812, 2.228, 2.764, 3.169, 3.581, 4.144, 4.587),
    ELEVEN(11, 0.697, 1.363, 1.796, 2.201, 2.718, 3.106, 3.497, 4.025, 4.437),
    TWELVE(12, 0.695, 1.356, 1.782, 2.179, 2.681, 3.055, 3.428, 3.93, 4.318),
    THIRTEEN(13, 0.694, 1.35, 1.771, 2.16, 2.65, 3.012, 3.372, 3.852, 4.221),
    FOURTEEN(14, 0.692, 1.345, 1.761, 2.145, 2.624, 2.977, 3.326, 3.787, 4.14),
    FIFTEEN(15, 0.691, 1.341, 1.753, 2.131, 2.602, 2.947, 3.286, 3.733, 4.073),
    SIXTEEN(16, 0.69, 1.337, 1.746, 2.12, 2.583, 2.921, 3.252, 3.686, 4.015),
    SEVENTEEN(17, 0.689, 1.333, 1.74, 2.11, 2.567, 2.898, 3.222, 3.646, 3.965),
    EIGHTEEN(18, 0.688, 1.33, 1.734, 2.101, 2.552, 2.878, 3.197, 3.61, 3.922),
    NINETEEN(19, 0.688, 1.328, 1.729, 2.093, 2.539, 2.861, 3.174, 3.579, 3.883),
    TWENTY(20, 0.687, 1.325, 1.725, 2.086, 2.528, 2.845, 3.153, 3.552, 3.85),
    TWENTY_ONE(21, 0.686, 1.323, 1.721, 2.08, 2.518, 2.831, 3.135, 3.527, 3.819),
    TWENTY_TWO(22, 0.686, 1.321, 1.717, 2.074, 2.508, 2.819, 3.119, 3.505, 3.792),
    TWENTY_THREE(23, 0.685, 1.319, 1.714, 2.069, 2.5, 2.807, 3.104, 3.485, 3.768),
    TWENTY_FOUR(24, 0.685, 1.318, 1.711, 2.064, 2.492, 2.797, 3.091, 3.467, 3.745),
    TWENTY_FIVE(25, 0.684, 1.316, 1.708, 2.06, 2.485, 2.787, 3.078, 3.45, 3.725),
    TWENTY_SIX(26, 0.684, 1.315, 1.706, 2.056, 2.479, 2.779, 3.067, 3.435, 3.707),
    TWENTY_SEVEN(27, 0.684, 1.314, 1.703, 2.052, 2.473, 2.771, 3.057, 3.421, 3.69),
    TWENTY_EIGHT(28, 0.683, 1.313, 1.701, 2.048, 2.467, 2.763, 3.047, 3.408, 3.674),
    TWENTY_NINE(29, 0.683, 1.311, 1.699, 2.045, 2.462, 2.756, 3.038, 3.396, 3.659),
    THIRTY(30, 0.683, 1.31, 1.697, 2.042, 2.457, 2.75, 3.03, 3.385, 3.646),
    THIRTY_ONE(31, 0.682, 1.309, 1.696, 2.04, 2.453, 2.744, 3.022, 3.375, 3.633),
    THIRTY_TWO(32, 0.682, 1.309, 1.694, 2.037, 2.449, 2.738, 3.015, 3.365, 3.622),
    THIRTY_THREE(33, 0.682, 1.308, 1.692, 2.035, 2.445, 2.733, 3.008, 3.356, 3.611),
    THIRTY_FOUR(34, 0.682, 1.307, 1.091, 2.032, 2.441, 2.728, 3.002, 3.348, 3.601),
    THIRTY_FIVE(35, 0.682, 1.306, 1.69, 2.03, 2.438, 2.724, 2.996, 3.34, 3.591),
    THIRTY_SIX(36, 0.681, 1.306, 1.688, 2.028, 2.434, 2.719, 2.99, 3.333, 3.582),
    THIRTY_SEVEN(37, 0.681, 1.305, 1.687, 2.026, 2.431, 2.715, 2.985, 3.326, 3.574),
    THIRTY_EIGHT(38, 0.681, 1.304, 1.686, 2.024, 2.429, 2.712, 2.98, 3.319, 3.566),
    THIRTY_NINE(39, 0.681, 1.304, 1.685, 2.023, 2.426, 2.708, 2.976, 3.313, 3.558),
    FORTY(40, 0.681, 1.303, 1.684, 2.021, 2.423, 2.704, 2.971, 3.307, 3.551),
    FIFTY(50, 0.679, 1.299, 1.676, 2.009, 2.403, 2.678, 2.937, 3.261, 3.496),
    SIXTY(60, 0.679, 1.296, 1.671, 2, 2.39, 2.66, 2.915, 3.232, 3.46),
    SEVENTY(70, 0.678, 1.294, 1.667, 1.994, 2.381, 2.648, 2.899, 3.211, 3.436),
    EIGHTY(80, 0.678, 1.292, 1.664, 1.99, 2.374, 2.639, 2.887, 3.195, 3.416),
    NINETY(90, 0.677, 1.291, 1.662, 1.987, 2.368, 2.632, 2.878, 3.183, 3.402),
    ONE_HUNDRED(100, 0.677, 1.29, 1.66, 1.984, 2.364, 2.626, 2.871, 3.174, 3.39),
    TWO_HUNDRED(200, 0.676, 1.286, 1.653, 1.972, 2.345, 2.601, 2.839, 3.131, 3.34),
    FIVE_HUNDRED(500, 0.675, 1.283, 1.648, 1.965, 2.334, 2.586, 2.82, 3.107, 3.31),
    ONE_THOUSAND(1000, 0.675, 1.282, 1.646, 1.962, 2.33, 2.581, 2.813, 3.098, 3.3),
    MAX(10000, 0.6745, 1.2816, 1.6449, 1.96, 2.3263, 2.5758, 2.807, 3.0902, 3.2905);

    /**
     * 自由度
     */
    private final int rof;
    /**
     * p0.5
     */
    private final double zeroDotFive;
    /**
     * p.0.2
     */
    private final double zeroDotTwo;
    /**
     * p.0.1
     */
    private final double zeroDotOne;
    /**
     * p.0.05
     */
    private final double zeroDotZeroFive;
    /**
     * p.0.02
     */
    private final double zeroDotZeroTwo;
    /**
     * 0.01
     */
    private final double zeroDotZeroOne;
    /**
     * 0.005
     */
    private final double zeroDotZeroZeroFive;
    /**
     * 0.002
     */
    private final double zeroDotZeroZeroTwo;
    /**
     * 0.001
     */
    private final double zeroDotZeroZeroOne;


    /**
     * 构造方法
     *
     * @param rof 显示内容
     */
    TDistributionEnum(int rof, double zeroDotFive, double zeroDotTwo, double zeroDotOne, double zeroDotZeroFive, double zeroDotZeroTwo, double zeroDotZeroOne, double zeroDotZeroZeroFive, double zeroDotZeroZeroTwo, double zeroDotZeroZeroOne) {
        this.rof = rof;
        this.zeroDotFive = zeroDotFive;
        this.zeroDotTwo = zeroDotTwo;
        this.zeroDotOne = zeroDotOne;
        this.zeroDotZeroFive = zeroDotZeroFive;
        this.zeroDotZeroTwo = zeroDotZeroTwo;
        this.zeroDotZeroOne = zeroDotZeroOne;
        this.zeroDotZeroZeroFive = zeroDotZeroZeroFive;
        this.zeroDotZeroZeroTwo = zeroDotZeroZeroTwo;
        this.zeroDotZeroZeroOne = zeroDotZeroZeroOne;
    }


    /**
     * 获取p值对应的T值
     *
     * @param rof 自由度
     * @return
     */
    public static TDistributionEnum getValue(int rof) {
        for (TDistributionEnum c : TDistributionEnum.values()) {
            if (c.getRof() == rof) {
                return c;
            }
        }
        return null;
    }


    public int getRof() {
        return rof;
    }

    public double getZeroDotFive() {
        return zeroDotFive;
    }

    public double getZeroDotTwo() {
        return zeroDotTwo;
    }

    public double getZeroDotOne() {
        return zeroDotOne;
    }

    public double getZeroDotZeroFive() {
        return zeroDotZeroFive;
    }

    public double getZeroDotZeroTwo() {
        return zeroDotZeroTwo;
    }

    public double getZeroDotZeroOne() {
        return zeroDotZeroOne;
    }

    public double getZeroDotZeroZeroFive() {
        return zeroDotZeroZeroFive;
    }

    public double getZeroDotZeroZeroTwo() {
        return zeroDotZeroZeroTwo;
    }

    public double getZeroDotZeroZeroOne() {
        return zeroDotZeroZeroOne;
    }
}
```

