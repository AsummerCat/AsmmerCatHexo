---
title: 26.深度学习-tf.keras实现
date: 2026-03-02 14:37:44
tags: [机器学习]
---

# tf.keras实现

跟sklearn类似,但是不支持接受`字符串型`的标签,需要热编码
不同的是:

*   构建分类器时需要进行模型搭建
*   数据采集时,sklearn可以接受字符串型的标签,如`setosa`,但是在kf.keras中需要对标签值进行热编码

## 1.热编码方式

有很多种热编码方式 ,比如pandas中的`get_dummies()`

<!--more-->

*   使用`kf.keras`中的方法进行热编码

<!---->

    # 数据处理的辅助工具
    from tensorflow.kears import utils

    # 进行热编码
    def one_hot_encode_object_array(arr):
        # 去重获取全部的类别
        uniques,ids=np.unique(arr,return_inverse=true)
        # 返回热编码的结果
        return utils.to_categorical(ids,len(uniques))
        
        
    # 数据处理
    ## 对标签值进行热编码
    ### 训练集热编码
    train_y_che =one_hot_encode_object_array(train_y)
    ### 测试集热编码
    train_y_che =one_hot_encode_object_array(test_y)



## 2.模型搭建  利用sequential方式创建模型

在`sklearn`中,模型都是现成的. tf.keras需要手动构建
以隐藏层通常使用`relu`

*   简单模型使用`Sequential`进行构建
*   复杂模型使用函数式编程来构建

```
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



# 通过model.summary可以查看模型的架构
model.summary()

# 通过utils.plot_model可以查看模型的架构
utils.plot_model(model,show_shapes=True)

```

## 3.模型训练和预测

#### 3.1 优化策略和损失函数

    # 设置模型的相关参数: 优化器,损失函数loss和评价指标  
    ## categorical_crossentropy 交叉熵验证 
    model.compile(optimizer='adam',loss='categorical_crossentropy',metrics=["accuracy"])

#### 3.2 模型训练

    # 模型训练: enochs,训练样本送入到网络中的次数,batch_size:每次训练的送入到网络中的样本个数 ,verbose:是否打印执行过程
    model.fit(train_X,rain_y_ohe,epochs=10,batch_size=1,verbose=1)

#### 3.3 模型评估

类似sklearn.score方法

    # 计算模型的损失和准确率
    loss,accuracy=model.evaluate(test_X,test_y_ohe,verbose=1)
    print("输出准确率:",accuracy)

