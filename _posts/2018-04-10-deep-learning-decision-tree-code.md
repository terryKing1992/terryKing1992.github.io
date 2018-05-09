---
layout: post
title: 机器学习算法之决策树算法代码实现
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
        shanno_entropy = shanno_entropy - prob * log(prob, 2)
        
    return shanno_entropy

# 后面我们模拟一个简单的数据, 数据中总共有两个分类,分别是'1', '2';
dataset = [[2, 3, 4, '1'], [1, 3, 5, '2'], [5, 4, 1, '1'], [1, 3, 4, '2'], [4, 5, 9, '2']]
print(cal_shannon_entropy(dataset))
```

最后我们可以该数据集上的信息熵为: ```0.9709505944546686```

当我们人为的增加另外一个标签数据的时候, 我们可以看到熵的变化

```python
dataset = [[2, 3, 4, '1'], [1, 3, 5, '2'], [5, 4, 1, '1'], [1, 3, 4, '2'], [4, 5, 9, '2'], [11, 9, 12, '3']]
print(cal_shannon_entropy(dataset))
```

我们可以看到该数据集上的信息熵为: ```1.0114042647073516```

得到熵之后, 我们就可以按照最大信息增益的方法来划分数据集了。

如果我们要向得到最大信息增益的话, 同样我们需要知道以某一个特征作为分割数据的标准后, 对于切割出来的数据的信息熵是多少.
那么我们就要一个个的算出来, 在某一特征下在切割完了的数据集上的信息熵是多少.

按照前面的方法
```
1、我们先将数据通过某一个特征切割开来
2、然后分别计算切割出来的数据集的信息熵
3、然后乘以 切割数据集在源数据集的占比
4、最后将切割出来的数据集上的信息熵相加即可
```

我们先开发代码能够把数据集进行切割

```python
# 根据某一列中的标签为$value 进行数据集的划分
### 比如我们有一系列的数据如下
### 身高   年龄    体重    是否活泼  标签: 是否打篮球
### 身高有3个选项, 高 中 低
### 年龄: 老年, 中年, 青年
### 体重: 肥胖, 标准, 偏瘦
### 是否活泼: 活泼, 不活泼

# 那么这个value就表示每一个标准的不同取值
def split_dataset(dataset, axis, value):
    axis_data = dataset[axis]
    
    split_data = []
    for row_data in dataset:
        if row_data[axis] == value:
            row_data_without_axis_data = []
            row_data_without_axis_data.extend(row_data[:axis])
            row_data_without_axis_data.extend(row_data[axis + 1:])
            split_data.append(row_data_without_axis_data)
            
    return split_data
```

下面我们构造一点真实数据:


| ID  |身高    | 年龄  | 体重 | 是否活泼| 是否打篮球  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |高    | 老年    | 标准  | 是  |  是  |
| 2   |高    | 中年    | 肥胖 | 否  |  否  |
| 3   |高    | 青年    | 偏瘦 | 是  |  是  |
| 4   |中    | 中年    | 偏瘦 | 是  |  否  |
| 5   |中    | 青年   |  标准  | 是  |  是  |
| 6   |低    | 青年    | 标准  | 否  |  否  |
| 7   |中    | 青年    | 肥胖  | 否  |  否  |
| 8   |低    | 老年    | 偏瘦  | 否  |  否  |
| 9   |低    | 青年   | 标准 | 是  |  是  |

我们有了数据, 然后在初始化一个Python数组, 我们就可以在这个数据集上进行试验了

```python
dataset = [['高', '老年', '标准', '是' ,'是'],
           ['高', '中年', '肥胖', '否' ,'否'],
           ['高', '青年', '偏瘦', '是' ,'是'],
           ['中', '中年', '偏瘦', '是' ,'否'],
           ['中', '青年', '标准', '是' ,'是'],
           ['低', '青年', '标准', '否' ,'否'],
           ['中', '青年', '肥胖', '否' ,'否'],
           ['低', '老年', '偏瘦', '否' ,'否'],
           ['低', '青年', '标准', '是' ,'是']
          ]
print(split_dataset(dataset, 0, '高'))
```

那我们就可以依据上面写的分割数据集的函数来对数据进行切割了.

那么我们先在第一列的数据上根据```高``` ```中``` ```低```属性值来切分数据得到如下3个子数据集

```
[['老年', '标准', '是', '是'], ['中年', '肥胖', '否', '否'], ['青年', '偏瘦', '是', '是']]
```

```
[['中年', '偏瘦', '是', '否'], ['青年', '标准', '是', '是'], ['青年', '肥胖', '否', '否']]
```

```
[['青年', '标准', '否', '否'], ['老年', '偏瘦', '否', '否'], ['青年', '标准', '是', '是']]
```

然后分别计算各个子数据集上的信息熵是多少

```python
original_entropy = cal_shannon_entropy(dataset)
print("最开始的信息熵为:" + str(original_entropy))
high_entropy = cal_shannon_entropy(split_dataset(dataset, 0, '高'))
middle_entropy = cal_shannon_entropy(split_dataset(dataset, 0, '中'))
low_entropy = cal_shannon_entropy(split_dataset(dataset, 0, '低'))
print(high_entropy)
print(middle_entropy)
print(low_entropy)
```

得到如下结果:
```
最开始的信息熵为0.6869615765973234
0.6365141682948128
0.6365141682948128
0.6365141682948128
```

同时, 因为每个子数据集占总数据集的1/3, 所以最终的```身高```这个特征作为分割数据的标准的话, 得到的信息增益为:

```
Gain = 0.6869615765973234 - (1/3 * 0.6365141682948128 + 1/3 * 0.6365141682948128 + 1/3 * 0.6365141682948128)
     = 0.0504474083025106
```

后面再写程序直接求出来应该以哪个axis(列)为特征才能得到最大的信息增益.