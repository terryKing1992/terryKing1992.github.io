---
layout: post
title: 机器学习算法之决策树算法
date: 2018-05-07 22:30:00
tags: [机器学习, 决策树算法]
---

### 前言
前面的文章中也提到了关于决策树算法的ID3算法原理, 下面逐步使用Python实现一个决策树算法.

首先, 根据前面的理论我们知道, 如果要构建一棵决策树那么需要先知道原始数据集的熵是多少；我们先看一下如何实现数据集上熵的计算.

假设我们的数据格式最后一列表示数据的真实标签列; 那么我们可以定义代码如下:

```python
import numpy as np
from math import log

# 计算数据集上的信息熵
def cal_shannon_entropy(dataset):
    # 知道数据集的大小
    dataset_count = len(dataset)
    
    # 负责存储数据集中的 "标签: 标签出现次数"
    labels_count_dict = {}
    
    for data_item in dataset:
        # 数据集中的每一行最后一个元素就是标签类型
        label = data_item[-1]
        
        # 如果存在就将标签出现的次数加一
        if label not in labels_count_dict:
            labels_count_dict[label] = 0
        labels_count_dict[label] = labels_count_dict[label] + 1
        
    # 由信息熵公式我们可以得到以下代码:
    shanno_entropy = 0.0
    for label in labels_count_dict:
        prob = float(labels_count_dict[label] / dataset_count)
        shanno_entropy = shanno_entropy - prob * log(prob)
        
    return shanno_entropy

# 后面我们模拟一个简单的数据, 数据中总共有两个分类,分别是'1', '2';
dataset = [[2, 3, 4, '1'], [1, 3, 5, '2'], [5, 4, 1, '1'], [1, 3, 4, '2'], [4, 5, 9, '2']]
print(cal_shannon_entropy(dataset))
```

最后我们可以该数据集上的信息熵为: ```0.67301```
