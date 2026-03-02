---
title: 27.深度学习-神经网络ANN
date: 2026-03-02 14:39:45
tags: [机器学习]
---

# 神经网络(ANN)

是一种模仿生物神经网络结构和功能的`计算模型`

*   特点: 不需要手动设计特征,可解释性差,效果好

*   层次: 分为三层 `输入层`,`隐藏层`,`输出层`

## 1.神经元工作方式

人工神经元接受到一个输入或者多个输入,对他们进行加权并相加,总和通过一个非线性函数产生输出.

<!--more-->


## 2.激活函数

需要选择非线性函数

    比如 sigmoid /logistics函数

    一般来说sigmoid网络在4层之内就会产生梯度消失的现象,而且,该激活函数并不是以0为中心的,
    所以在实践中这种激活函数使用的很少.sigmoid函数一般只用于二分类的输出层

#### 2.1 sigmoid函数

*   实现方法:
    直接使用tf实现sigmoid

```
import tensorflow as tf
import tensorflow.keras as keras
import matplotlib.pyplot as plt
import numpy as np

# 定义x的取值范围
x=np.linspace(-10,10,100)
# 直接使用tensorflow实现
y=tf.nn.sigmoid(x)
# 绘图
plt.plot(x,y)
plt.grid()

```

#### 2.2 tanh(双曲正切曲线)

一种非常常见的激活函数

它是以0为中心的,使得其收敛速度要比`sigmoid`快,减少迭代次数.也是会出现梯度为0的情况
若使用时可在隐藏层使用`tanh`函数,在输出层使用sigmoid函数

```
import tensorflow as tf
# import tensorflow.keras as keras
import matplotlib.pyplot as plt
import numpy as np

# 定义x的取值范围
x=np.linspace(-10,10,100)
# 直接使用tensorflow实现
y=tf.nn.tanh(x)
# 绘图
plt.plot(x,y)
plt.grid()

```

#### 2.3 relu激活函数

ReLU是目前最常见的激活函数.
可以无脑使用

```
import tensorflow as tf
# import tensorflow.keras as keras
import matplotlib.pyplot as plt
import numpy as np

# 定义x的取值范围
x=np.linspace(-10,10,100)
# 直接使用tensorflow实现
y=tf.nn.relu(x)
# 绘图
plt.plot(x,y)
plt.grid()

```

#### 2.4 LeakReLu激活函数

是ReLU激活函数的改进.

```
import tensorflow as tf
# import tensorflow.keras as keras
import matplotlib.pyplot as plt
import numpy as np

# 定义x的取值范围
x=np.linspace(-10,10,100)
# 直接使用tensorflow实现
y=tf.nn.leaky_relu(x)
# 绘图
plt.plot(x,y)
plt.grid()

```

#### 2.4 SoftMax激活函数

作用于多分类过程中 ,他是二分类函数sigmoid在多分类上的推广,目的是将多分类的结果以概率的形式展示出来
预测目标的类别

```
import tensorflow as tf
# import tensorflow.keras as keras
import matplotlib.pyplot as plt
import numpy as np

# 数字中的score
x=tf.constant([0.2,0.02,0.15,1.3,0.5,0.06,1.1,0.05,3.75])
# 将其送入softmax中计算分类结果
y=tf.nn.softmax(x)
# 将结果进行打印
print(y)

```

#### 2.5 激活函数的选择

隐藏层:

*   优先选择RELU激活函数
*   如果relu效果不好,那么尝试其他激活函数,如Leaky Relu等
*   如果你选择了relu,需要注意一下dead relu问题,避免出现大的梯度从而导致过多的神经元死亡
*   不要使用sigmoid激活函数,可以尝试使用tanh激活函数

输出层:

*   二分类问题选择sigmoid激活函数
*   多分类问题选择softmax激活函数
*   回归问题选择identity激活函数

## 3. 参数初始化

对于某一个神经元来说,需要初始化的参数有两类: 一类是权重W,还有一类是偏置b,偏置b初始化为0即可.而权重W的初始化比较重要

#### 3.1 随机初始化 (基本很少用了)

随机初始化从均值0,标准差是1的高斯分布中取值,使用一些很小的值对参数W进行初始化

#### 3.2 标准初始化

权重参数初始化从区间均匀随机取值, 即在均匀分布中生当前神经元的权重,其中d为每个神经元的输入数量

#### 3.3 Xavier初始化 (现阶段使用比较多) 默认使用

该方法的基本实现就是各层的激活值和梯度的方差在传播过程中保持一致.也叫做`Glorot初始化`

*   正态化Xavier初始化: `glorot_normal`
    正态分布的随机中选择

<!---->

    import tensorflow as tf
    #进行实例化
    initializer=tf.keras.initializers.glorot_normal()
    # 取样得到权重值
    values=initializer(shape=(9,1))
    # 打印结果
    print(values)

*   标准化Xavier初始化: `glorot_uniform`
    均匀分布中采样

<!---->

    import tensorflow as tf
    #进行实例化
    initializer=tf.keras.initializers.glorot_uniform()
    # 取样得到权重值
    values=initializer(shape=(9,1))
    # 打印结果
    print(values)

#### 3.4 He初始化 (常用)

也叫kaiming初始化

*   正态化He初始化: `he_normal`

```
import tensorflow as tf
#进行实例化
initializer=tf.keras.initializers.he_normal()
# 取样得到权重值
values=initializer(shape=(9,1))
# 打印结果
print(values)

```

*   标准化He初始化: `he_uniform`
    均匀分布中采样

<!---->

    import tensorflow as tf
    #进行实例化
    initializer=tf.keras.initializers.he_uniform()
    # 取样得到权重值
    values=initializer(shape=(9,1))
    # 打印结果
    print(values)

## 4. 损失函数

`损失函数越小,越接近真实结果`

用来衡量模型参数的质量的函数,衡量的方式是比较模型输出和真实值之间的差异
ps: 可能会被叫做: 误差,目标,代价函数

#### 4.1 分类任务的损失函数

在深度学习的分类任务中使用最多的是`交叉损失函数(CategoricalCrossentropy)`

*   多分类任务  损失函数:`CategoricalCrossentropy`

<!---->

    import tensorflow as tf
    # 设置真实值和预测值
    y_true=[[0,1,0],[0,0,1]]
    y_pred=[[0.05,0.95,0],[0.01,0.8,0.1]]
    # 实例化交叉熵损失
    cce=tf.keras.losses.CategoricalCrossentropy()
    #计算损失结果
    cce(y_true,y_pred).numpy()

*   二分类任务 损失函数:`BinaryCrossentropy`

<!---->

    import tensorflow as tf
    # 设置真实值和预测值
    y_true=[[0],[1]]
    y_pred=[[0.5],[0.6]]
    # 实例化交叉熵损失
    cce=tf.keras.losses.BinaryCrossentropy()
    #计算损失结果
    cce(y_true,y_pred).numpy()

#### 4.2 回归任务的损失函数

常见的损失函数     `MAE损失`,`MSE损失` 也就是所谓的L1,L2损失可以当做正则项来用

*   MAE损失(`MeanAbsoluteError`): L1损失  常用在正则项加在损失函数上

<!---->

    import tensorflow as tf
    # 设置真实值和预测值
    y_true=[[0.],[1.]]
    y_pred=[[1.],[0.]]
    # 实例化MAE L1损失函数
    cce=tf.keras.losses.MeanAbsoluteError()
    #计算损失结果
    cce(y_true,y_pred).numpy()

*   MSE损失(`MeanSquaredError`): L2损失  常用在正则化上

<!---->

    import tensorflow as tf
    # 设置真实值和预测值
    y_true=[[0.],[1.]]
    y_pred=[[1.],[0.]]
    # 实例化MSE L2损失函数
    cce=tf.keras.losses.MeanSquaredError()
    #计算损失结果
    cce(y_true,y_pred).numpy()

*   smooth(`Huber`) L1损失: 通常在目标检测中使用该损失函数   (回归基本上使用这个 兼具了L1,L2损失的特性)

<!---->

    import tensorflow as tf
    # 设置真实值和预测值
    y_true=[[0.],[1.]]
    y_pred=[[1.],[0.]]
    # 实例化smooth L1损失函数
    cce=tf.keras.losses.Huber()
    #计算损失结果
    cce(y_true,y_pred).numpy()

## 5.神经网络的搭建

`tf.Keras`中构建模有两种方式,一种是通过`Sequential`构建,一种是通过`Model`类构建.

*   前者是按一定的顺序对层进行堆叠
*   后者可以用来构建较复杂的网络模型

#### 5.1 通过Sequential构建

    import tensorflow as tf
    from tensorflow import keras
    from tensorflow.keras import layers

    # 利用sequential方式创建模型
    model = keras.Sequential([
        # 隐藏层1, 激活函数是relu, 输入大小由input_shape(特征维度)指定
        layers.Dense(10, activation="relu", input_shape=(4,)),
        # 隐藏层2, 激活函数是relu
        layers.Dense(10, activation="relu"),
        # 输出层
        layers.Dense(3, activation="softmax")
    ], name="我的测试神经元")

    # 打印模型结构
    model.summary()

    主要参数:
    utils: 当前层包含的神经元个数
    activation: 激活函数,relu,sigmoid等
    use_bias: 是否使用偏置,默认使用偏置
    Kernel_initializer: 权重的初始化方法,默认是Xavier初始化
    bias_initializer: 偏置的初始化方式,默认为0

#### 5.2 通过Model类构建 (Function API方式构建)

    import tensorflow as tf

    # 定义模型的输入
    inputs= tf.keras.Input(shape=(3,),name="input")
    # 第一层: 激活函数为relu,其他为默认
    x=tf.keras.layers.Dense(10, activation="relu",name="layer1")(inputs)
    # 第二层: 激活函数为relu,其他为默认
    x=tf.keras.layers.Dense(2, activation="relu",name="layer2")(x)
    # 第三层:(输出层) 激活函数为Sigmoid
    outputs=tf.keras.layers.Dense(2, activation="sigmoid",name="layer3")(x)
    # 使用Model来创建模型,指明输入和输出
    model=tf.keras.Model(inputs=inputs,outputs=outputs,name="根据Model创建模型")

    # 展示模型结果
    model.summary()

#### 5.3 通过Model的子类构建

通过model的子类构建模型,此时需要在\_init\_中定义神经网络的层,在call方法中定义网络的前向传播过程

    import tensorflow as tf

    # 定义model的子类
    class MyModel(tf.keras.Model):
        # 在init方法中定义网络的层次结构
        def __init__(self):
            super(MyModel,self).__init__()
            # 第一层:
            self.layer1=tf.keras.layers.Dense(10, activation="relu", input_shape=(4,))
            # 第二层:
            self.layer2=tf.keras.layers.Dense(2, activation="relu", input_shape=(4,))
            # 第三层: 输出层
            self.layer3=tf.keras.layers.Dense(2, activation="sigmoid")

        # 在call方法中完成前向传播    
        def call(self, inputs):
            x=self.layer1(inputs)
            x=self.layer2(x)
            return self.layer3(x)

    # 实例化模型
    model=MyModel()
    ## 设置一个输入,调用模型(否则无法使用summay)
    x=tf.ones((1,3))
    y=model(x)    

    model.summary()

## 6.神经网络的优缺点

*   优点:

<!---->

    1.精度高,性能由于其他的及其学习算法,甚至在某些领域超过了人类
    2.可以近似任意的非线性函数
    3.有大量的框架和库可以供调用

*   缺点:

<!---->

    1.黑盒,很难解释模型是怎么工作的
    2.训练时间唱,需要大量的计算力
    3.网络结构复杂,需要调整超参数
    4.小训练集上表现不佳,容易发生过拟合

