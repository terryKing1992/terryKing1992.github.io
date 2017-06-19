---
layout: post
title: 使用tensorflow api生成one-hot标签数据
date: 2017-06-29 22:50:01
tags: [tensorflow, one-hot]
---

在刚开始学习tensorflow的时候, 会有一个最简单的手写字符识别的程序供新手开始学习, 在tensorflow.example.tutorial.mnist中已经定义好了mnist的训练数据以及测试数据. 并且标签已经从原来的List变成了one-hot的二维矩阵的格式.看了源码的就知道mnist.input_data.read_data()这个方法中使用的是numpy中的方法来实现标签的one-hot矩阵化。那么如何使用tensorflow中自带的api来实现呢?下面我们就来一起看一下需要用到的api吧。

**tf.expand_dims 方法**
这个函数主要给矩阵或者数组增加一维°, 看代码可能更加清晰:

```python
import tensorflow as tf
# 比如现在有一个列表
x_data = [1, 2, 3, 4]
x_data_expand = tf.expand_dims(x_data, 0) # x_data的shape是[4], 该函数表示在最前面的位置增加一维, 就会变成[1, 4]
# 而对于[1, 4] 的矩阵加上x_data本身的数据, 那么可以猜想到x_data_expand = [[1, 2, 3, 4]]

x_data_expand_axis1 = tf.expand_dims(x_data, axis=1) # x_data的shape是[4], 而axis=1表示在本来的矩阵的第1列加一维, 所以x_data_expand_axis1是[4, 1] 4行一列的矩阵, 并且把原始数据套进去可知: x_data_expand_axis1 = [[1], [2], [3], [4]], 但是这个axis的参数值不能大于矩阵的列数, 比如矩阵shape为[1, 2, 3] 那么axis=0 则会生成[1, 1, 2, 3], axis=1则会生成[1, 1, 2, 3], axis=2则会生成[1, 2, 1, 3], axis=3则会生成[1, 2, 3, 1]。就是在某一个位置插入一列
```

**tf.concat(values, axis)**
该函数用于将两个相同维度的数据进行合并, 如果指定axis=0那么只需要列数相同即可.否则需要维度都相同 看如下代码:

```python
import tensorflow as tf
x_data = [[1, 2, 3], [4, 5, 6]]
y_data = [[7, 8, 9], [10, 11, 12]]

concat_result = tf.concat(values=[x_data, y_data], axis=0) # 这样的话, 生成的数据是[[1, 2, 3], [7, 8, 9], [4, 5, 6], [10, 11, 12]]
concat_result = tf.concat(values=[x_data, y_data], axis=1) # 这样的话, 生成的数据是[[ 1  2  3  7  8  9], [ 4  5  6 10 11 12]], 三维的甚至更高维度的数据稍后再尝试
```

**tf.sparse_to_dense()**
def sparse_to_dense(sparse_indices,
                    output_shape,
                    sparse_values,
                    default_value=0,
                    validate_indices=True,
                    name=None):
该函数指定位置赋值, 并且生成一个维度为output_shape的矩阵;如果output_shape维度为1, 那么sparse_indices只能是一个列表, 如果output_shape为二维矩阵, 那么sparse_indices就可以是矩阵了.

比如如下代码:

```python
import tensorflow as tf

sparse_indices = [1, 2, 6]
output_shape = tf.zeros([10]).shape
sparse_output = tf.sparse_to_dense(sparse_indices, output_shape, 2, default_value=0) # 生成的结果为:sparse_output:[0 2 2 0 0 0 2 0 0 0] 就是在位置1, 2, 6的位置填充2 其余位置填充0

# 对于二维矩阵的填充也是一样的, 比如:
sparse_indices = [[0, 1], [2, 4], [4 ,5], [6, 9]]
output_shape = tf.zeros([6, 10]).shape
sparse_output = tf.sparse_to_dense(sparse_indices, output_shape, 1, default_value=0) #生成的数据如下:# 生成的数据如下:sparse_output:
[[0 1 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 1 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 1 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 1]
 [0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]
 [0 0 0 0 0 0 0 0 0 0]]
```

**下面开始实现对原始标签列表的one-hot化**

```python
  import tensorflow as tf

  labels  = [1, 3, 4, 8, 7, 5, 2, 9, 0, 8, 7]
  labels_expand = tf.expand_dims(labels, axis=1) # 这样label_expand为[11, 1]的数据

  index_expand = tf.expand_dims(tf.range(len(labels)), axis=1) # 与label_expand中的元素一一对应

  concat_result = tf.concat(values=[index_expand, labels_expand], axis=1) # 将上述两组数据组合在一起

  one_hot = tf.sparse_to_dense(sparse_indices=concat_result, output_shape=tf.zeros([len(labels), 10]).shape, sparse_values=1.0, default_value=0.0)

  session = tf.InteractiveSession()

  print('labels_expand:{}'.format(session.run(labels_expand)))
  print('index_expand:{}'.format(session.run(index_expand)))

  print('concat_result:{}'.format(session.run(concat_result)))
  print('one_hot_of_labels:{}'.format(session.run(one_hot)))
```

最后的结果如下打印:

  labels_expand:[[1]
  [3]
  [4]
  [8]
  [7]
  [5]
  [2]
  [9]
  [0]
  [8]
  [7]]
  index_expand:[[ 0]
  [ 1]
  [ 2]
  [ 3]
  [ 4]
  [ 5]
  [ 6]
  [ 7]
  [ 8]
  [ 9]
  [10]]
  concat_result:[[ 0  1]
  [ 1  3]
  [ 2  4]
  [ 3  8]
  [ 4  7]
  [ 5  5]
  [ 6  2]
  [ 7  9]
  [ 8  0]
  [ 9  8]
  [10  7]]
  one_hot_of_labels:[[ 0.  1.  0.  0.  0.  0.  0.  0.  0.  0.]
  [ 0.  0.  0.  1.  0.  0.  0.  0.  0.  0.]
  [ 0.  0.  0.  0.  1.  0.  0.  0.  0.  0.]
  [ 0.  0.  0.  0.  0.  0.  0.  0.  1.  0.]
  [ 0.  0.  0.  0.  0.  0.  0.  1.  0.  0.]
  [ 0.  0.  0.  0.  0.  1.  0.  0.  0.  0.]
  [ 0.  0.  1.  0.  0.  0.  0.  0.  0.  0.]
  [ 0.  0.  0.  0.  0.  0.  0.  0.  0.  1.]
  [ 1.  0.  0.  0.  0.  0.  0.  0.  0.  0.]
  [ 0.  0.  0.  0.  0.  0.  0.  0.  1.  0.]
  [ 0.  0.  0.  0.  0.  0.  0.  1.  0.  0.]]

这样就实现了labels的one-hot化。
