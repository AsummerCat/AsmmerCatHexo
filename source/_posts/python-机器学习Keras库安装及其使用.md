---
title: python-机器学习Keras库安装及其使用
date: 2020-01-13 22:35:45
tags: [python,机器学习]
---

# Keras库

人工神经网络

还有一个是google的TensorFlow库 两个常用

<!--more-->

https://blog.csdn.net/chenweifu365/article/details/79094710

## keras官网 http://leras.io

# [demo地址](https://github.com/AsummerCat/keras-demo)

## 安装

```python
windows:
Anaconda CMD
conda install mingw libpython
pip install keras

mac:
  conda install -c conda-forge keras
```



## 如果直接使用keras 而不使用TensorFlow的话 需要设置

```python
windows下:
C:\Users\用户名\keras\keras-json中
修改:
"image_dim_ordering": "th",
"teckend": "theano"
```



## 测试

```python
# -*- coding: utf-8 -*-
import numpy as np
# 神经网络的 各个层的容器
from keras.models import Sequential
# Dense 求和的层  Activation 激活函数
from keras.layers import Dense, Activation
# SGD 随机梯度下降算法
from keras.optimizers import SGD


def test():
    # 直接引入鸢尾iris数据集
    from sklearn.datasets import load_iris
    iris = load_iris()

    # 重新序列标签化 001 010 100
    from sklearn.preprocessing import LabelBinarizer
    LabelBinarizer().fit_transform(iris["target"])

    from sklearn.model_selection import train_test_split
    ## 将数据分为测试数据 和验证数据   test_size表示测试数据占比20% random_state=1 随机选择30个数据
    train_data, test_data, train_target, test_target = train_test_split(iris.data, iris.target, test_size=0.2,
                                                                        random_state=1)
    # 标签化处理
    labels_train = LabelBinarizer().fit_transform(train_target)
    labels_test = LabelBinarizer().fit_transform(test_target)

    # 构建神经网络层
    ## 方式一
    model = Sequential(
        [
            # 网络结构
            Dense(5, input_dim=4),
            Activation("relu"),
            Dense(3),
            Activation("sigmoid"),
        ]
    )

    ## 方式二
    # model = Sequential()
    # model.add(Dense(5,input=4))

    # 添加优化器
    sgd = SGD(LR=0.01, decay=1e-6, nesterov=True)
    ## 指定优化器,指示函数
    model.compile(optimizer=sgd, loss="categorical_crossentropy")

    # 训练      nb_epoch:训练多少轮  batch_size=训练一批需要多少数据
    model.fit(train_data, labels_train, nb_epoch=200, batch_size=40)

    # 预测
    print(model.predict_classes(test_data))

    # 保存训练参数 以便下次继续使用
    model.save_weights("./data/w")
    # 下次使用的话
    model.load_weights("./data/w")

    pass


if __name__ == '__main__':
    test()

```

