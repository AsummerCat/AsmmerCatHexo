---
title: 31.深度学习-卷积神经网络CNN
date: 2026-03-02 14:43:47
tags: [机器学习]
---

# 卷积神经网络(CNN)

使用场景: 计算机视觉使用最多

为什么不使用ANN(全连接神经网络)?
会存在以下问题

```
1.需要处理的数据量大,效率低
2.图像在维度调整的过程中很难保留原有的特征,导致图像处理的准确率不高

```
<!--more-->

## 1.卷积神经网络(CNN)构成

卷积神经网络的工作流程可以概括为以下几个步骤：

卷积层：用卷积核在图像上滑动，提取特征。    <font color="red">(局部特征提取) </font>

池化层：减少数据的维度，保留关键特征。 <font color="red">(降维,防止过拟合)</font>
多层卷积：通过多层卷积和池化，逐步提取更高级的特征。
全连接层：将特征图展平成一维向量，进行分类。 (识别 检测)
损失函数和优化：通过损失函数和反向传播算法，不断调整模型的权重，提高预测准确性。

## 2.卷积层

卷积层是卷积神经网络中的核心模块,卷积层的目的是提取输入特征图的特征,卷积核可以提取图像中的边缘信息

在kf.keras中卷积核的实现使用

    import tensorflow as tf

    tf.keras.layers.Conv2D(filters,kernel_size,strides=(1,1),padding='valid',activation=None)

    参数: 
    filters: 卷积核过滤波器的数量,对应输出特征图的通道数
    kernel_size: 过滤器filter的大小
    strides: 步长
    padding: valid:(在输入周围不进行padding); same: (padding后使输出特征图和输入特征图形状相同)
    activation: 激活函数

## 3.池化层(Pooling)

池化层迎来降低了后续网络层的输入维度,缩减模型大小,提高计算速度,防止过拟合

*   最大池化
    取窗口内的最大值作为输出

<!---->

    import tensorflow as tf
    # 最大池化
    tf.keras.layers.MaxPool2D(pool_size=(2,2),strides=None,padding='valid')
    # 参数
    # poolsize: 池化窗口大小
    # strides: 窗口移动的步长,默认为1
    # padding: 是否进行填充,默认不进行填充          

*   平均池化
    取窗口内的所有值的均值作为输出

<!---->

    import tensorflow as tf
    # 平均池化
    tf.keras.layers.AveragePooling2D(pool_size=(2,2),strides=None,padding='valid')
    # 参数
    # poolsize: 池化窗口大小
    # strides: 窗口移动的步长,默认为1
    # padding: 是否进行填充,默认不进行填充         

## 4.全连接层

卷积层(特征提取)+池化(降维),之后将特征图转换为一维向量送入到全连接层进行分类或回归的操作

使用 `tf.keras.dense`实现
