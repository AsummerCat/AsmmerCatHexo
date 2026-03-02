---
title: 30.深度学习-demo简单案例模拟
date: 2026-03-02 14:43:15
tags: [机器学习]
---

# 案例

# 1.案例流程

*   数据加载
*   数据处理
*   模型构建
*   模型训练
*   模型测试
*   模型保存

<!--more-->

# 2.导入所需的工具包

```
import numpy as np
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize']=(7,7) #画板大小

import tensorflow as tf
#数据集
from tensorflow.keras.datasets import mnist
# 构建序列模型
from tensorflow.keras.models import Sequential
# 导入需要的层
from tensorflow.keras.layers import Dense,Dropout, Activation,BatchNormalization
# 导入辅助工具包
from tensorflow.keras import utils
# 正则化
from tensorflow.keras import regularizers


```



# 3.数据加载

```
# 1.数据加载
## 类别总数
nb_classes=10
## 加载数据集
(X_train,y_train),(X_test,y_test)=mnist.load_data()
## 打印输出数据集的维度
print("训练样本初始维度",X_train.shape)
print("训练样本目标值初始维度",y_train.shape)

## 数据展示: 将训练集的前九个数据集进行展示
for i in range(9):
    plt.subplot(3,3,i+1)
    #以灰度图展示,不进行插值
    plt.imshow(X_train[i],cmap='gray',interpolation='none')
    # 设置图片的标题: 对应的类别
    plt.title("数字{}".format(y_train[i]))

```

# 4.数据处理

神经网络中的每个训练样本室一个向量,所以需要对输入进行重塑,使每个28\*28的图像成为一个784维向量,将输入的数据进行归一化处理,从0-255调整为0-1

```
# 2.数据处理
## 调整数据维度: 每一个数字转换成一个向量
X_train=X_train.reshape(60000,784)
X_test=X_test.reshape(10000,784)
## 格式转换 浮点数类型可以更好地进行数值计算，尤其是在归一化和梯度下降等操作中。
X_train=X_train.astype('float32')
X_test=X_test.astype('float32')
## 归一化
# X_train/=255
# X_test/=255
# 使用 tf.keras.utils.normalize 进行归一化
X_train = tf.keras.utils.normalize(X_train, axis=-1)
X_test = tf.keras.utils.normalize(X_test, axis=-1)

## 维度调整后的结果
print("训练集",X_train.shape)
print("测试集",X_test.shape)

## 目标值进行热编码 num_classes：类别数量。如果未指定，则会自动推断为 y 中的最大值加1。
y_train=tf.keras.utils.to_categorical(y_train,nb_classes, dtype='float32')
y_test=tf.keras.utils.to_categorical(y_test,nb_classes, dtype='float32')

```

# 5.模型构建

```
# 3.创建模型
## 利用序列模型来构建模型
model=Sequential()
## 全连接层,共512个神经元,输入维度大小维784
model.add(Dense(512,activation="relu",input_shape=(784,)))
## 使用正则化方法
model.add(tf.keras.layers.Dropout(0.2))
## 全连接层,共512个神经元,并加入L2正则化
model.add(Dense(512,kernel_regularizer=regularizers.l2(0.001)))
## BN层
model.add(BatchNormalization())
#激活函数
model.add(Activation('relu'))
model.add(tf.keras.layers.Dropout(0.2))
## 全连接层 ,输出层共10个神经元
model.add(Dense(10))
## softmax将神经网络输出的score转换为概率值
model.add(Activation('softmax'))
## 查看结果
model.summary()

```

# 6.模型编译

    model.compile(optimizer='adam',loss='categorical_crossentropy',metrics=["accuracy"])

# 7.模型训练

```
# 5.模型训练
history = model.fit(
    X_train,  # 输入数据
    y_train,      # 目标数据
    epochs=4,                      # 最大训练轮数
    batch_size=128,                   # 批量大小
    verbose=1,                       # 输出训练过程
    validation_data=(X_test,y_test)  #模型会使用验证数据评估性能。这有助于监控模型的泛化能力，防止过拟合。
)

# 6.将损失函数绘制成曲线
plt.figure()
# 训练集损失函数变换
plt.plot(history.history["loss"],label="train_loss")
# 验证集损失函数变化
plt.plot(history.history["val_loss"],label="val_loss")
plt.legend()
plt.grid()

# 7.准确率
plt.figure()
plt.plot(history.history["accuracy"],label="train")
plt.plot(history.history["val_accuracy"],label="val")
plt.legend()
plt.grid()



```

# 8.模型测试

    # 8.测试模型
    score=model.evaluate(X_test,y_test,verbose=1)
    ## 打印测试结果 损失率与准确率
    print('测试集准确率:',score)

# 9.模型保存

```

# 9.模型保存
## 保存模型架构和权重在h5文件中
model.save('model.h5')
## 加载模型:包含架构和对应的权重
model=tf.keras.models.load_model('model.h5')
```

# 10.完整代码

```
import numpy as np
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize']=(7,7) #画板大小
plt.rcParams['font.sans-serif'] = ['SimHei']  # 指定默认字体
plt.rcParams['axes.unicode_minus'] = False    # 解决保存图像是负号'-'显示为方块的问题

import tensorflow as tf
#数据集
from tensorflow.keras.datasets import mnist
# 构建序列模型
from tensorflow.keras.models import Sequential
# 导入需要的层
from tensorflow.keras.layers import Dense,Dropout, Activation,BatchNormalization
# 导入辅助工具包
from tensorflow.keras import utils
# 正则化
from tensorflow.keras import regularizers



# 1.数据加载
## 类别总数
nb_classes=10
## 加载数据集
(X_train,y_train),(X_test,y_test)=mnist.load_data()
## 打印输出数据集的维度
print("训练样本初始维度",X_train.shape)
print("训练样本目标值初始维度",y_train.shape)

## 数据展示: 将训练集的前九个数据集进行展示
for i in range(9):
    plt.subplot(3,3,i+1)
    #以灰度图展示,不进行插值
    plt.imshow(X_train[i],cmap='gray',interpolation='none')
    # 设置图片的标题: 对应的类别
    plt.title("数字{}".format(y_train[i]))

# 2.数据处理
## 调整数据维度: 每一个数字转换成一个向量
X_train=X_train.reshape(60000,784)
X_test=X_test.reshape(10000,784)
## 格式转换 浮点数类型可以更好地进行数值计算，尤其是在归一化和梯度下降等操作中。
X_train=X_train.astype('float32')
X_test=X_test.astype('float32')
## 归一化
# X_train/=255
# X_test/=255
# 使用 tf.keras.utils.normalize 进行归一化
X_train = tf.keras.utils.normalize(X_train, axis=-1)
X_test = tf.keras.utils.normalize(X_test, axis=-1)
## 维度调整后的结果
print("训练集",X_train.shape)
print("测试集",X_test.shape)
## 目标值进行热编码 num_classes：类别数量。如果未指定，则会自动推断为 y 中的最大值加1。
y_train=tf.keras.utils.to_categorical(y_train,nb_classes, dtype='float32')
y_test=tf.keras.utils.to_categorical(y_test,nb_classes, dtype='float32')


# 3.创建模型
## 利用序列模型来构建模型
model=Sequential()
## 全连接层,共512个神经元,输入维度大小维784
model.add(Dense(512,activation="relu",input_shape=(784,)))
## 使用正则化方法
model.add(tf.keras.layers.Dropout(0.2))
## 全连接层,共512个神经元,并加入L2正则化
model.add(Dense(512,kernel_regularizer=regularizers.l2(0.001)))
## BN层
model.add(BatchNormalization())
#激活函数
model.add(Activation('relu'))
model.add(tf.keras.layers.Dropout(0.2))
## 全连接层 ,输出层共10个神经元
model.add(Dense(10))
## softmax将神经网络输出的score转换为概率值
model.add(Activation('softmax'))
## 查看结果
model.summary()


# 4.模型编译 优化器,损失函数,评价指标
model.compile(optimizer='adam',loss='categorical_crossentropy',metrics=tf.keras.metrics.Accuracy())

# 5.模型训练
history = model.fit(
    X_train,  # 输入数据
    y_train,      # 目标数据
    epochs=4,                      # 最大训练轮数
    batch_size=128,                   # 批量大小
    verbose=1,                       # 输出训练过程
    validation_data=(X_test,y_test)  #模型会使用验证数据评估性能。这有助于监控模型的泛化能力，防止过拟合。
)

# 6.将损失函数绘制成曲线
plt.figure()
## 训练集损失函数变换
plt.plot(history.history["loss"],label="train_loss")
## 验证集损失函数变化
plt.plot(history.history["val_loss"],label="val_loss")
plt.legend()
plt.grid()

# 7.准确率
plt.figure()
plt.plot(history.history["accuracy"],label="train")
plt.plot(history.history["val_accuracy"],label="val")
plt.legend()
plt.grid()


# 8.测试模型 
score=model.evaluate(X_test,y_test,verbose=1)
## 打印测试结果 损失率与准确率
print('测试集准确率:',score)

# 9.模型保存
## 保存模型架构和权重在h5文件中
model.save('model.h5')
## 加载模型:包含架构和对应的权重
model=tf.keras.models.load_model('model.h5')


```

