---
layout: post
title: 深度学习之循环神经网络
date: 2018-04-10 07:40:00
tags: [深度学习, 循环神经网络]
---

### 前言

循环神经网络RNN相对来说是一个很简单的网络结构, 网上也有大把的文章来介绍RNN到底是什么东西?RNN能干什么, 但是由于我一直纠结于RNN前一个节点的输出与后一个节点的输入应该以一种什么样的连接方式进行连接的问题一直不得要领, 窥不见RNN的真实面目. 通过昨天看Tensorflow的BasicRNNCell的源码实现才自认为是懂了一点RNN的基本原理与实现.


### RNN的定义

RNN循环神经网络能够记忆前一个输入的一些信息, 能够解决长期依赖问题：在语言模型、语音识别中需要根据上下文进行推断和预测，上下文的获取可以根据马尔科夫假设获取固定上下文。RNN可以通过中间状态保存上下文信息，作为输入影响下一时序的预测。当然也可以不用理会关于RNN的这些概念性质的东西. 

对于RNN结构的展开还是要介绍一下的, 虽然网上一大堆相关图片.

![RNN结构展开形状](/assets/images/2018-04-09-deeplearning-rnn-unfold.png)

将上图中的展开式表示成函数形式就是这个样子的

![RNN的函数表现](/assets/images/2018-04-09-deep-learning-rnn-math.png)

这种函数的表现形式基本上是一个通用的函数表示方式, 当然我们可以在该函数内部进行数据的不同组合. 我原来就是迷失在了对于前面的输出h<sub>t-1</sub> 到底应该以一种怎么样的方式与后面x<sub>t</sub>进行拼接. 这里有两种方式一种是相加的方式 一种是拼接的方式(当然也肯定有其他方式我还没发现而已). 后面的代码中会使用Tensorflow展示这两种实现方式. 这里使用Tensorflow的目的是为了第一、可以直接使用BasicRNNCell类 第二、可以省去自己写反向传播的过程。

### 代码展示

首先, 我们直接使用Tensorflow里面已经实现的BasicRNNCell来实现图片分类的功能. 假设我们的图片是1通道, 28*28的数据。那么每一行都可以看做是不同时刻的输入, 总共有28个时刻. 因为一个图像的形成是由于前后的像素之间的关联才生成的, 还是可以使用RNN来完成的。

对于手写字符识别的训练数据, 大家可以去

[MNIST_data下载地址](http://yann.lecun.com/exdb/mnist/)

```python
import numpy as np
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data

mnist = input_data.read_data_sets('MNIST_data', one_hot=True)

# 每一行的大小作为时刻t的输入, 如果加上batch_size的话. 那么在t时刻神经网络的输入应该是[batch_size, input_size]
input_size = 28
# 因为每一行都可以看作是不同时刻的输入, 所以对于一张图片总共要输入28次, 才能把图片数据用完
time_steps = 28

# 每次训练的数据量大小
batch_size = 64

# 作为前一个神经元的输出的维度, 目前定义为256维, 再加上batch_size.
#  那么每输入[batch_size, input_size]的数据得到的传到下一个时刻的输出应该为[batch_size, hidden_size]
hidden_size = 256

# 最终的分类为10分类问题
n_classes = 10

# 最长运行多少步
max_steps = 10000

# 定义网络所需要的输入, None表示batch_size, time_steps表示整个RNN中有28个相同的Cell连接而成, input_size表示每个时刻的输入大小
input_x = tf.placeholder(dtype=tf.float32, shape=[None, time_steps, input_size])

# 作为评估模型准确度的标签数据
y_label = tf.placeholder(dtype=tf.float32, shape=[None, 10])

# 最后一个全连接层, 对应于最后一个Cell的output所需要的WB
output_weights = tf.Variable(tf.random_normal(shape=[hidden_size, n_classes], dtype=tf.float32))
output_bias = tf.Variable(tf.random_normal(shape=[n_classes], dtype=tf.float32))

# 定义一个基本的RNNCell
rnn_cell = tf.nn.rnn_cell.BasicRNNCell(num_units=hidden_size)

# 使用Tensorflow自带的RNNCell的连接, 求值方式大大降低了用户构建RNN的复杂度, time_major表示第1维的数据是否是时间序列上面的数据
# 这里很显然我们可以知道, 第一位上面是batch_size, 而时间序列数据在第二维上面(也就是图片的28行数据)。所以这里我们选择否
outputs, states = tf.nn.dynamic_rnn(rnn_cell, input_x, dtype=tf.float32, time_major=False)

# 经过一个全连接层, 得到最后的分类, 这里之所以要将output旋转一下是因为 我们从上面RNN得到的输出shape为[batch_size, time_steps, hidden_size]
# 所以需要将第一维数据与第二维数据交换一下, 才能得到我们最后想要的[batch_size, hidden_size]数据
logits = tf.nn.xw_plus_b(tf.transpose(outputs, [1, 0, 2])[-1], output_weights, output_bias)

cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=logits, labels=y_label))

train_op = tf.train.AdamOptimizer().minimize(cost)

accuracy = tf.reduce_mean(tf.cast(tf.equal(tf.argmax(logits, axis=1), tf.argmax(y_label, axis=1)), tf.float32))

with tf.Session() as session:
    session.run(tf.global_variables_initializer())

    for index in range(max_steps):
        batch_images, batch_labels = mnist.train.next_batch(batch_size)
        reshape_images = batch_images.reshape([-1, time_steps, input_size])
        session.run(train_op, feed_dict={input_x: reshape_images, y_label: batch_labels})

        if index % 100 == 0:
            accuracy_result = session.run(accuracy, feed_dict={input_x: mnist.test.images.reshape([-1, time_steps, input_size]), y_label: mnist.test.labels})
            print('训练' + str(index) + '步后, 测试集上精度为' + str(accuracy_result))
```

Tensorflow的BasicRNNCell默认采用的是Linear类来将t-1时刻Cell的输出 拼接t时刻的输入作为t时刻真实的输入来完成的.

在我们上面这个程序中, 因为我们的上个节点的输出维度为[64, 256], 而下个节点的输入维度为[64, 28]. tensorflow的Linear类把这两个数据进行了拼接得到了一个[64, 284]的数据, 然后乘以了一个[284, 256]的Weights最终得到了t时刻的输出.

当然可能Tensorflow里面还要其他的拼接方式, 后面我也会自己写一个向量相加的方式完成RNN的实现.

明天继续更新。
