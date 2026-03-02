---
title: 25.深度学习-TensorFolw框架
date: 2026-03-02 14:37:09
tags: [机器学习]
---

# 1.TensorFolw框架

这是一个深度学习框架,`在计算机视觉,音频处理,推荐系统和自然语言处理等场景下都被大面积推广使用`

目前已经有2.8版本对应python3.10 及其以上

提供了可视化分析工具`Tensorboard`方便分析和调整模型

*   基础工作流程:
*   1.使用tf.data加载数据. (实例化读取训练数据和测试数据)
*   2.模型的建立和调试: 使用动态图模式`Eager Execution`和著名的神经网络高层API框架`Keras`,结合可视化工具`TensorBoard`,建立,快速的建立和调试模型
*   3.模型的训练: 支持CPU/单GPU/单机多卡GPU/多机集群/TPU训练模型,充分利用海量数据和计算资源进行高效训练
*   4.预训练模型调用: 通过`TensorFlow Hub`,可以方便的调用预训练完毕的已有成熟模型
*   5.模型的部署: 通过`TensorFlow Serving ,TensorFlow Lite ,TensorFlow.js`等组件,可以将`TensorFlow`模型部署到服务器,移动端,嵌入式端等多种使用场景

<!--more-->

## 2.安装

```
CPU:
pip install tensorflow
GPU: 
pip install tensorflow-gpu


conda install tensorflow
conda install tensorflow-gpu
conda install -c conda-forge tensorflow
我的是3.10版本py

```

## 3. rf的 Tensor张量及其操作

张量是一个多维数组,`跟np的ndarray对象类似`,tf.Tensor对象也具有数据类型和形状

此外`tf.Tensor`可以保存在GPU中.`tensorflow`提供了丰富的操作库(`tf.add`,`tf.matmul`,`tf.linalg.inv`等)

#### 3.1 导入对应工具包

```
import tensorflow as tf
import numpy as np

```

#### 3.2 基础方法->张量 tf.constant

    import tensorflow as tf
    import numpy as np


    # 基础方法

    ## 创建int32类型的0维张量,即标量
    rank_0_tensor=tf.constant(4)
    print(rank_0_tensor)

    ## 创建float32类型的1维张量
    rank_1_tensor=tf.constant([2.0,3.0,4.0])
    print(rank_1_tensor)

    ## 创建float16类型的2维张量
    rank_2_tensor=tf.constant([[2.0,3.0],[2.0,3.0],[2.0,3.0]],dtype=tf.float16)
    print(rank_2_tensor)


    tf.Tensor(4, shape=(), dtype=int32)
    tf.Tensor([2. 3. 4.], shape=(3,), dtype=float32)
    tf.Tensor(
    [[2. 3.]
     [2. 3.]
     [2. 3.]], shape=(3, 2), dtype=float16)
     
     输出表示 shape=(3, 2) 3X2的矩阵 也就是二维

*   转换为ndarray

<!---->

    import tensorflow as tf
    import numpy as np

    a=tf.constant([[1,2],[3,4]])

    # 转换 
    b=a.numpy()
    print(b)
    ## 或者 p.array
    c=np.array(a)
    print(c)

*   基础数学运算

```
import tensorflow as tf
import numpy as np
# 定义张量 a和b
a=tf.constant([[1,2],[3,4]])
b=tf.constant([[1,1],[1,1]])

# 计算张量的和
print(tf.add(a,b)) 
# 计算张量的元素乘法
print(tf.multiply(a,b)) 
# 计算张量的矩阵乘法
print(tf.matmul(a,b)) 

# 求和
tf.reduce_sum(a)
# 平均值
tf.reduce_mean(a)
# 最大值
tf.reduce_max(a)
# 最小值
tf.reduce_min(a)
# 最大值的索引
tf.argmax(a)
# 最小值索引
tf.argmin(a)

```

#### 3.3 基础方法->变量 tf.Variable

变量是一种特殊的张量,形状是不可变的,但可以更改其中改的参数

```
import tensorflow as tf
import numpy as np
# 定义变量
a=tf.Variable([[1,2],[3,4]])

# 修改里面的数值
a.assign(([[6,7],[8,9]]))

## 我们也可以获取它的形状,类型及转换为ndarray
print("Shape:",a.shape)
print("DType:",a.dtype)
print("AS numpy:",a.numpy)

```

## 4.tf.keras中的相关模块及其常用方法

`tf.keras`是高阶API接口,大大提升了TF代码的简洁性和复用性

#### 4.1 常用模块

| 模块            | 概述                 |
| ------------- | ------------------ |
| activations   | 激活函数               |
| applications  | 预训练网络模块            |
| callbacks     | 在模型训练期间被调用         |
| datasets      | tf.keras数据集模块      |
| layers        | kears层API          |
| losses        | 损失函数               |
| metircs       | 各种评价指标             |
| models        | 模型创建模块,以及与模型相关的API |
| optimizers    | 优化方法               |
| preprocessing | keras数据的预处理模块      |
| regularizers  | 正则化L1,L2等          |
| utils         | 辅助模块实现             |

#### 4.2 使用

*   导入tf.keras

<!---->

    import tensorflow as tf
    from tensorflow import kears

配置训练过程:

*   配置训练过程:

```
# 配置优化方法,损失函数和评价指标
model.compile(optimizer=tf.train.AdamOptimizer(0.001),
              loss='categorical_crossentropy',
              metircs=['accuracy'])

```

*   模型训练:

<!---->

    # 指明训练数据集,训练epoch,批次大小和验证集数据
    model.fit/fit_generator(dataset,epochs=10,
                           batch_size=3,
                           validation_data=val_dataset
                           )

*   模型评估

<!---->

    # 指明评估数据集和批次大小
    model.evaluate(x,y,batch_size=32)

*   模型预测

<!---->

    # 对新的样本进行预测
    model.predict(x,batch_size=32)

*   回调函数

```
用来控制模型训练过程中的回调函数
也可以使用tf.kears.callbacks内置的callback:

ModelCheckpoint:  定期保存检查点
LearningRateScheduler: 动态改变学习速率
EarlyStopping: 当验证集上的性能不再提高时,终止训练.
TensorBoard:  使用TensorBoard检测模型的状态

```

*   模型的保存和恢复

<!---->

    # 只保存模型的权重
    model.save_weights('/models')
    # 加载模型的权重
    model.load_weights('/models')

    # 保存整个模型架构与权重在h5文件中
    model.save('model.h5')

    # 加载模型(包括 架构和权重)
    model=kears.models.load_model('model.h5')

## 5.测试

    # 用于模型搭建
    from tensorflow.kears.models import Sequential
    # 构建模型的层级和激活方法
    from tensorflow.kears.layers import Dense,Activation
    # 数据处理的辅助工具
    from tensorflow.kears import utils

