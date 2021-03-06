---
layout: post
title: 机器学习算法之决策树算法
date: 2018-04-05 21:40:00
tags: [机器学习, 决策树算法]
---

## 定义

决策树是一种典型的分类方法, 首先我们对数据进行整理，利用归纳算法生成可读的规则, 然后利用生成的规则对未知数据进行预测。本质上来说决策树就是利用一些规则对数据进行分类的过程。

## 决策树相关算法

1. Hunt，Marin 和 Stone于1996年提出的CLS学习系统, 用于学习单个概念

2. 1979年, J.R.Quinlan提出的ID3算法, 并对ID3进行总结和简化使得ID3算法成为决策树算法中的典型

3. 1988年, Utgoff在ID4的基础上提出了ID5算法, 进一步提高了决策树的效率

4. 1993年 Quinlan进一步改进了ID3算法, 改进成了C4.5算法

5. 另一类决策算法为CART

## ID3算法

在本章中, 我们先了解一下ID3算法是如何进行的, ID3算法是针对于属性选择问题, 是决策树中最具影响力和最为典型的算法。该方法使用信息增益的方式进行节点的选择.

那么信息增益应该怎么算呢? 在了解信息增益如何计算之前, 我们还需要引入另外一个基本概念叫做信息熵. 熵定义为信息的期望值. 我们通过如下公式来计算熵:

![香农熵计算公式](/assets/images/2018-04-05-machine-learning-entropy.png)

对于在某一个特征上的信息增益如何计算呢?

    Gain(A) = Info(D) - Info_A(D)

Info(D)表示在训练数据D集合信息熵, Info_A(D)表示在已知特征A的情况下的D集合信息熵. 这种表示确实不是一般人能理解的。
变量的不确定性越高, 信息熵越大, 而反过来如果变量的不确定性越低, 信息熵越小. 那么如果以A变量来划分数据, 如果A的确定性越高, 其在数据集上计算出来的熵越小, 那么其信息增益越大. 其实归根结底就是找能够使得分类更明确的特征。

下面我们用一个很简答的例子来说明如何计算一个特征在整个集合上的信息增益

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |青年    | 高    | 否  | 良  |  否  |
| 2   |青年    | 高    | 否  | 优  |  否  |
| 3   |中年    | 高    | 否  | 良  |  是  |
| 4   |老年    | 中    | 否  | 良  |  是  |
| 5   |老年    | 低    | 是  | 良  |  是  |
| 6   |老年    | 低    | 是  | 优  |  否  |
| 7   |中年    | 低    | 是  | 优  |  是  |
| 8   |青年    | 中    | 否  | 良  |  否  |
| 9   |青年    | 低    | 是  | 良  |  是  |

首先, 我们要选择一个特征作为决策树的根节点。当然我们首先要知道在不使用任何特征的前提下, 该数据集上的信息熵为多少?

根据是否购买的情况，利用上面我们给出的公式我们知道

![原始数据的信息熵](/assets/images/2018-04-05-machine-learning-info-d.png)

其中, 因为没有考虑任何的特征, 我们知道在原始数据中, 买电脑的占 5/9 而不买电脑的人占总数的 4/9. 所以得到了上面的数据。

随后我们根据年龄将数据切分为三部分： 青年, 中年, 老年

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |青年    | 高    | 否  | 良  |  否  |
| 2   |青年    | 高    | 否  | 优  |  否  |
| 8   |青年    | 中    | 否  | 良  |  否  |
| 9   |青年    | 低    | 是  | 良  |  是  |

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 3   |中年    | 高    | 否  | 良  |  是  |
| 7   |中年    | 低    | 是  | 优  |  是  |

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 4   |老年    | 中    | 否  | 良  |  是  |
| 5   |老年    | 低    | 是  | 良  |  是  |
| 6   |老年    | 低    | 是  | 优  |  否  |

同样, 我们依据上面的公式可以有如下推理过程:

![以年龄来分类计算得出的信息熵](/assets/images/2018-04-05-machine-learning-info-a.png)

我们对三部分的数据分别求信息熵, 然后再乘以该部分占总数据的比例.这样我们就知道了年龄这个特征的信息增益为: 

```0.33```

依据公式: Gain(A) = Info(D) - Info_A(D)

同样的方法, 我们以收入切割数据将数据分为三部分: 高, 中, 低

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |青年    | 高    | 否  | 良  |  否  |
| 2   |青年    | 高    | 否  | 优  |  否  |
| 3   |中年    | 高    | 否  | 良  |  是  |

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 8   |青年    | 中    | 否  | 良  |  否  |
| 4   |老年    | 中    | 否  | 良  |  是  |

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 9   |青年    | 低    | 是  | 良  |  是  |
| 7   |中年    | 低    | 是  | 优  |  是  |
| 5   |老年    | 低    | 是  | 良  |  是  |
| 6   |老年    | 低    | 是  | 优  |  否  |

根据上面的公式可以有如下推理过程:

![以收入来分类计算得到的信息熵](/assets/images/2018-04-08-machine-learning-entropy-with-salary.png)

依据公式: Gain(A) = Info(D) - Info_A(D) 我们知道在已知收入特征的情况下, 在数据集的分类上表现出来的不确定性(信息熵)为 ```0.89```

所以, 数据分类在```收入```这个特征上的的信息增益为 ```0.1```

为了大家更加容易理解怎么计算的, 下面我们继续以学生特征为例, 来求信息熵

我们根据是否为学生的特征将数据集分为如下两部分:

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |青年    | 高    | 否  | 良  |  否  |
| 2   |青年    | 高    | 否  | 优  |  否  |
| 3   |中年    | 高    | 否  | 良  |  是  |
| 8   |青年    | 中    | 否  | 良  |  否  |
| 4   |老年    | 中    | 否  | 良  |  是  |

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 9   |青年    | 低    | 是  | 良  |  是  |
| 7   |中年    | 低    | 是  | 优  |  是  |
| 5   |老年    | 低    | 是  | 良  |  是  |
| 6   |老年    | 低    | 是  | 优  |  否  |

根据上面的公式可以有如下推理过程:

![以是否为学生计算得到的信息熵](/assets/images/2018-04-08-machine-learning-entropy-with-isa-student.png)

所以, 数据分类在```是否为学生```这个特征上的的信息增益为 ```0.12```

下面我们根据信誉特征将数据集分为以下两部分:

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |青年    | 高    | 否  | 良  |  否  |
| 9   |青年    | 低    | 是  | 良  |  是  |
| 3   |中年    | 高    | 否  | 良  |  是  |
| 8   |青年    | 中    | 否  | 良  |  否  |
| 4   |老年    | 中    | 否  | 良  |  是  |
| 5   |老年    | 低    | 是  | 良  |  是  |

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 2   |青年    | 高    | 否  | 优  |  否  |
| 7   |中年    | 低    | 是  | 优  |  是  |
| 6   |老年    | 低    | 是  | 优  |  否  |


根据上面的公式可以有如下推理过程:

![以是否为学生计算得到的信息熵](/assets/images/2018-04-08-machine-learning-entropy-with-credit.png)


所以, 数据分类在```信誉```这个特征上的的信息增益为 ```0.07```

因为使用年龄特征作为根节点能够将分类的熵(不确定性)快速的减到最小, 所以根节点应该选择```年龄```作为根节点来构建决策树.

依据年龄作为根节点之后, 我们还要往下构建决策树, 那么就需要知道到底选哪个作为三个分支(青年、中年、老年)的根节点

对于分组后的数据集数据集, 本来应该将年龄特征从该数据集中删除掉的, 但是为了查看方便, 我还是选择保留在了分组后的数据集中.
但是在分别计算数据子集的熵的时候不应该把年龄特征再考虑进去了.

于是, 我们有了如下的三组数据:

青年数据集:

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 1   |青年    | 高    | 否  | 良  |  否  |
| 2   |青年    | 高    | 否  | 优  |  否  |
| 8   |青年    | 中    | 否  | 良  |  否  |
| 9   |青年    | 低    | 是  | 良  |  是  |

中年数据集:

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 3   |中年    | 高    | 否  | 良  |  是  |
| 7   |中年    | 低    | 是  | 优  |  是  |

老年数据集:

| ID  |年龄    | 收入  | 学生 | 信誉| 是否购买电脑  |
| -   |-----  | ----  | ----| -: | :----: |
| 4   |老年    | 中    | 否  | 良  |  是  |
| 5   |老年    | 低    | 是  | 良  |  是  |
| 6   |老年    | 低    | 是  | 优  |  否  |

我们可以再次使用上面的方式进行特征的选择. 可以计算得出

在青年数据集上, 如果不叠加任何特征, 那么数据集分类上面的熵为: 0.81

在青年数据集上, 信息增益(收入) = 0.81

在青年数据集上, 信息增益(学生) = 0.81

在青年数据集上, 信息熵(信誉) = 0.69

在青年数据集上, 信息增益(信誉) = 0.81 - 0.69 = 0.12

所以, 在决策树的第二层青年分支上面, 选择收入或者学生这两个特征中的任意一个都可以作为决策节点。

在中年数据集上没什么好说的. 信息熵直接=0, 所以节点直接退出了.

在老年数据集上, 我们同样可以计算得出:

在不叠加任何特征的情况下, 数据分类的信息熵为 0.92

信息增益(收入) = 0.92 - 0.66 = 0.26
信息增益(学生) = 0.92 - 0.66 = 0.26
信息增益(信誉) = 0.92 - 0 = 0.92

所以在老年分支上面, 直接使用信誉特征作为判断条件即可.

到这里依托于上面的数据, 我们就将决策树构建出来了.

当然我们最终构建的决策树如图所示:

![最终的决策树](/assets/images/2018-04-08-machine-learning-decision-tree-final-png.png)

这个决策树构建出来看着不符合常理, 那是因为数据集是我自己瞎编出来的.

如果一个低收入的青年的数据过来, 经过我们的决策树进行决策那么出来的结果就是他会买电脑. 当然在超级简单的数据集上只是为了理解决策树的构建过程. 对于复杂的决策树理论后面再详细学习。
