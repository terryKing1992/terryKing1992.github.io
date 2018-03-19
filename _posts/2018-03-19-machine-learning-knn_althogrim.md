---
layout: post
title: 机器学习算法之K近邻算法
date: 2018-03-19 21:00:00
tags: [机器学习, K近邻算法]
---

在做机器学习方面的工程的时候, 动手实践才是能够快速掌握的正确方法, 在后面的日子里, 我会慢慢的学习基础的机器学习方法以及数学理论, 争取让自己做到懂理论的实践主义者.
从今天开始, 首先学习第一个最最最基础的算法, K近邻算法.

K近邻算法(K-Near-Neighbor)起的名字还是很容易让人知道它的算法原理的. 在对于分类问题上, 我们可以通过一个距离比较算法, 算出待预测的数据与已知数据之间的距离, 然后选择距离最近的k个数据, 最后再比较这K个数据里面哪个分类最多, 预测结果就是算出来的分类. 这个真的是最基本的机器学习算法了, 我们完全不需要公式就可以理解意思, 并且还能够编写源码出来完成这项工作, 并不需要一些成熟的机器学习框架. 我们是自己开发的噢, 这样才能更加深刻的理解算法。

当然上面提到的距离公式也有很多, 在本篇中, 我们使用欧式距离公式:

![欧式距离公式](/assets/images/2018-03-19-machine-learning-knn.png)

例如: 当我们的训练数据集中的特征只有两个的时候比如下面提到的坐标点(1, 1), (2, 4) 之间的欧式距离就可以这么算:

![两个特征的欧氏距离](/assets/images/2018-03-19-machine-learning-knn-example1.png)

如果表示三维坐标体系或者训练样本中有3个特征的时候计算方式如下:

![三个特征的欧氏距离](/assets/images/2018-03-19-machine-learning-knn-example.png)

K近邻算法一般的流程如下:

```
1、收集数据: 可以使用任意方法
2、准备数据: 距离计算所需要的数值, 最好是结构化的数据格式
3、分析数据: 可以使用任意方法
4、训练数据: 在KNN算法上不适用
5、测试宣发: 计算错误率
6、使用算法: 使用该算法对未知数据进行预测。
```



下面我们假设一个场景, 并且在该场景下使用KNN算法来预测未知数据.

### 场景一、预测某一个二维坐标位于哪个象限

哈哈, 可能有些人看到这个标题会忍不住大笑, 这TM还用机器学习方法? 我看(x, y)的符号就知道在哪个象限了啊. 同志, 你要知道要学习最高深的武功必须忘记前面学过的任何知识才能很快的接收新的知识噢, 想想张无忌吧~

当然在开始搭建模型之前, 我们还是要构造一些数据的, 构造的这些数据还是需要我们以前学过的知识的.
这个数据怎么生成呢? 我大概的思路是这样的

```
1、随机生成一堆有符号整数
2、将该整数两两结合, 生成一个[N, 2] N行两列的数组
3、同时利用我们已知的知识, 在这[N, 2] 后面再加一列 该坐标对应的象限, 在坐标轴上的数据舍弃, 那么我们的数据就变成了[N, 3]的数组
4、编写KNN算法, 通过传入预测参数, 来预测坐标应该属于哪个象限.
```

首先声明, 我并不怎么会用Python的API, 所以要查一下Python 随机生成数字怎么使用

```python
    import random
    random.randint(-100, 100)
```
我们利用上面的代码在区间[-100, 100]生成一个整数

如果想生成500个数字, 那么就需要调用该方法500次, 同时我们还不想要```x```或者```y```是```0```的数据. 那么我们要怎么生成呢?

```python
    # _*_ coding:utf-8 _*_

    import random

    def generate_knn_data():
        count = 0
        points = []
        while True:
            x = random.randint(-100, 100)
            y = random.randint(-100, 100)
            if x != 0 and y != 0:
                count = count + 1

            points.append(x)
            points.append(y)

            if count >= 500:
                break
        return points

    print(generate_knn_data())
```

现在我们生成了一个具有1000个非0数字的列表, 那么如何将数据整合成一个[N, 2]的数组呢? 我听说过一个Numpy的框架能够帮我们完成这个事情, 但是之前必须使用```pip install numpy -U```来安装该python包.

安装好了之后, 我们随便找个搜索引擎, 搜索 ```python list 转 numpy 数组``` 

```python
    import numpy as np

    num_list = [0, 1, 2, 3, 5]
    num_array = np.array(num_list)
    print(num_array)
```

我们使用上面代码, 先将python里的List转化为Numpy的数组, 有的人可能会说, 转这个干吗, 因为Numpy对于数组的操作函数比较多, 不用我们敲很多的代码. 我们用搜索到的知识来完成我们的任务.

```python
    # _*_ coding:utf-8 _*_

    import random
    import numpy as np


    def generate_knn_data():
        count = 0
        points = []
        while True:
            x = random.randint(-100, 100)
            y = random.randint(-100, 100)
            if x != 0 and y != 0:
                count = count + 1

            points.append(x)
            points.append(y)

            if count >= 500:
                break
        return points

    num_array = np.array(generate_knn_data())
    print(num_array)
```

通过上面的代码, 我们可以得到一个numpy的数组了, 后面就更加方便我们操作了, 首先 我们需要将numpy一维数组转变为二维数组

同时对各个训练数据进行标签

```python
    //将数据转为二维数组
    train_data = np.array(generate_knn_data()).reshape((-1, 2))
    print(train_data)


    def label(item):
        if item[0] > 0 and item[1] > 0:
            return 1
        elif item[0] > 0 and item[1] < 0:
            return 4
        elif item[0] < 0 and item[1] > 0:
            return 2
        else:
            return 3

    labels = np.array(list(map(label, train_data)))
    //对各个数据分别打标签
    print(labels)
```

下面完成我们思路里面的, 将train_data 与 labels数据进行融合, 形成一个[N, 3]的数组, 我们经查找发现numpy中有个```np.concatenate((a,b),axis=1)```将[N, 2] 和 [N, 1]拼接起来

```python
    final_train_datas = np.concatenate((train_data, labels), axis=1) 
```

最终我们将数据拼接成了如下的样子:

```
[[ 99 -40   4]
 [ 78  -3   4]
 [ 64  35   1]
 ..., 
 [-90  94   2]
 [ 81 -77   4]
 [ 25  86   1]]
```
后面我们需要封装一下我们的函数, 最终得到的结果如下:

```python
    # _*_ coding:utf-8 _*_

    import random
    import numpy as np


    def generate_knn_data():
        points = generate_points_data()
        labels = generate_labels_data(points)
        return np.concatenate((points, labels.reshape((-1, 1))), axis=1)


    def generate_points_data():
        count = 0
        points = []
        while True:
            x = random.randint(-100, 100)
            y = random.randint(-100, 100)
            if x != 0 and y != 0:
                count = count + 1

            points.append(x)
            points.append(y)

            if count >= 500:
                break
        return np.array(points).reshape((-1, 2))


    def generate_labels_data(points):
        def label_lambda(item):
            if item[0] > 0 and item[1] > 0:
                return 1
            elif item[0] > 0 and item[1] < 0:
                return 4
            elif item[0] < 0 and item[1] > 0:
                return 2
            else:
                return 3
        return np.array(list(map(label_lambda, points)))

    print(generate_knn_data())
```
