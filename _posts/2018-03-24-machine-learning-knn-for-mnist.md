---
layout: post
title: 机器学习算法之K近邻算法
date: 2018-03-24 22:00:00
tags: [机器学习, K近邻算法, 手写字符识别]
---

上篇文章使用点的象限分类来说明了对于KNN的基本用法, 为了巩固KNN的理论, 我们从另外一个较为使用的例子来说明KNN的用法.
对于MNIST数据集, 应该都不陌生, 号称是机器学习界的HelloWorld程序.

MNIST的目标是要解决一个手写数字识别的10分类问题. 既然是分类问题, 我们同样可以使用KNN算法来解决这个问题..

## 准备数据

我们可以从如下网址下载本篇文章所需要的训练数据

[digit数组地址](http://www.terrylmay.com/assets/datas/digits.zip)

## 开发KNN算法

如上篇同样的道理, 我们先读取训练数据以及训练数据对应的标签

```python
# _*_ coding: utf-8 _*_
import os

def generate_data(data_file):

    if not data_file.startswith('/'):
        data_file = os.path.join(os.path.dirname(os.path.realpath(__file__)), data_file)

    print(data_file)
    train_data_labels = []
    train_data_digits = []
    for digit_item in os.listdir(data_file):
        # 首先把标签读取出来
        label = int(digit_item.split('_')[0])
        train_data_labels.append(label)
        digit_file_path = os.path.join(data_file, digit_item)
        with open(digit_file_path, mode='r+') as digit_file:
            digit_content = []

            lines = digit_file.read().split('\n')

            for line in lines:
                digit_content.extend([int(char) for char in line])

        train_data_digits.append(digit_content)

    return train_data_digits, train_data_labels

train_data, train_labels = generate_data('digits/trainingDigits/')
print(train_data[0])
print(train_labels[0])
```

下面我们需要用同样的方式读取测试数据集以及label

```python
test_data, test_labels = generate_data('digits/testDigits/')
```

现在我们得到了测试集与数据集, 下面我们就可以通过KNN来判断测试集的标签是什么了, 并且通过与真实的测试标签对比可以知道我们的KNN精确度是多少？

现在我们就可以用到上篇文章中写的predict方法, 稍微改造一下就可以使用了. 最终的代码版本如下:

```python
# _*_ coding: utf-8 _*_

import os
import operator
import numpy as np


def read_digit_and_label(current_path, digit_item):
    # 首先把标签读取出来
    label = int(digit_item.split('_')[0])
    digit_file_path = os.path.join(current_path, digit_item)
    with open(digit_file_path, mode='r+') as digit_file:
        digit_content = []

        lines = digit_file.read().split('\n')

        for line in lines:
            digit_content.extend([int(char) for char in line])

    return digit_content, label


def generate_data(data_file):

    if not data_file.startswith('/'):
        data_file = os.path.join(os.path.dirname(os.path.realpath(__file__)), data_file)

    print(data_file)
    train_data_labels = []
    train_data_digits = []
    for digit_item in os.listdir(data_file):
        digit_content, label = read_digit_and_label(data_file, digit_item)
        train_data_labels.append(label)
        train_data_digits.append(digit_content)

    return np.asarray(train_data_digits), np.asarray(train_data_labels)

train_data, train_labels = generate_data('digits/trainingDigits/')
print(train_data[0])
print(train_labels[0])

test_data, test_label = generate_data('digits/testDigits/')
print(test_data[0])
print(test_label[0])


def predict(predict_data=None, train_datas=None, train_labels=None):
    train_data_size = train_datas.shape[0]
    predict_data_metrix = np.tile(predict_data, (train_data_size, 1))
    minus_result = predict_data_metrix - train_datas
    square_result = np.square(minus_result)
    sum_result = np.sum(square_result, axis=1)

    sqrt_result = np.sqrt(sum_result)
    sorted_distance = sqrt_result.argsort()

    sorted_dict = {}

    for index in range(10):
        label = train_labels[sorted_distance[index]]

        if label in sorted_dict:
            sorted_dict[label] += 1
        else:
            sorted_dict[label] = 1

    sorted_classes = sorted(sorted_dict.items(), key=operator.itemgetter(1), reverse=True)
    return sorted_classes[0][0]


test_data, test_label = read_digit_and_label(os.path.join(os.path.dirname(os.path.realpath(__file__)), 'digits/testDigits/'), digit_item='1_0.txt')

predict_result = predict(predict_data=test_data, train_datas=train_data, train_labels=train_labels)

print('预测值为:', predict_result, '真实值为:', test_label)

```

对于上面的结果我们只是一个个的预测测试集里面的数据分类, 如果想测试一下使用欧式距离的 5-NN算法的效果如何, 可以封装一下predict方法并且写个while循环即可;

```python

def accuracy(data_file):
    if not data_file.startswith('/'):
        data_file = os.path.join(os.path.dirname(os.path.realpath(__file__)), data_file)
    total_test_size = 0
    correct_result_size = 0
    for test_item in os.listdir(data_file):
        test_data, test_label = read_digit_and_label(data_file, digit_item=test_item)
        predict_result = predict(predict_data=test_data, train_datas=train_data, train_labels=train_labels)

        if predict_result == test_label:
            correct_result_size += 1
        total_test_size += 1

    return float(correct_result_size) / total_test_size


print('算法精度为:', accuracy('digits/testDigits/'))
```

这样我们可以看到输出如下, 其实算法的精度还是蛮高的:

```
/Users/terrylmay/Github/tensorflow_example/knn/digits/trainingDigits/
[0 0 0 ..., 0 0 0]
0
/Users/terrylmay/Github/tensorflow_example/knn/digits/testDigits/
[0 0 0 ..., 0 0 0]
0
算法精度为: 0.9799154334038055
```

基本上两个练习做完之后, 对于KNN的使用应该就已经入门了.