---
layout: post
title: 设计模式系列之访问者模式
date: 2018-01-10 20:43:00
tags: [设计准则, 访问者模式]
---

### 访问者模式

访问者模式: 封装一些作用于某种数据结构中的各个元素的操作, 他可以在不改变数据结构的前提下定义作用域这些元素的心得操作.这句话怎么理解呢? 简直不是人能够理解的了的, 这么抽象.有人曾经说过, 文章中增加一个公理或者定理或者数学公式, 那么你的读者就会降低40%。我们还是以很通俗的语言来说明访问者模式到底是用来干嘛的.

我的理解是: 1000个人眼中有1000个哈姆雷特. 怎么说呢? 每个人看到同一事物所表现出来的行为或者观点都是不一样的.

![访问者模式的通用类图](/assets/images/2018-01-10-design-pattern-visitor.png)

我们可以看到上面类图总共有4个核心类

1、Visitor: 定义访问者的行为

2、ConcreteVisitor: 具体的访问者实现

3、Element: 定义接收哪一类的访问者访问, 以及元素可能存在的一些抽象.

4、ConcreteElement: 定义具体的被访问者, 同时实现accept方法

5、ElementContainer: 有些地方也叫ObjectStructure, 不过按照我目前对访问者模式的理解, 我觉得叫Container可能更好理解一些。

### 应用场景

本身访问者模式在我见到的代码里面是不怎么常用的设计模式. 但是为了说明访问者模式到底可以在哪些场景下使用,我还是先列举几个场景. 然后在针对里面的某一个场景来开发代码, 加深对于访问者模式的记忆与使用.

场景一、在医院里, 当医生给我们开完处方之后, 通常需要到缴费处去交费. 缴费完成之后, 我们需要到药房去拿药. 那么我们可以将处方单理解为被访问对象, 而收费人员是一个访问者, 为我们准备药的人员是一个访问者. 收费人员关注处方下面的总价格. 而药房人员则关注都需要准备哪些药.

场景二、如果家里来了客人并且带着一个5岁大的小孩, 大人一进门可能会说你家布置的真漂亮, 而小孩则更加关注你家的玩具好多. 那么这个场景下房子就是一个被访问者, 而大人与小孩分别都是访问者. 

场景三、在看电影的时候, 前段时间有一部电影《战狼2》, 当看到最后一个镜头, 吴京把国旗套在胳膊上往前走的时候。很多人都会有一种民族自豪感以及莫名的感动. 但是也会有另外一少部分人觉得就会拿国人的民族自豪感来骗票房. 同一个镜头, 让不同的人看到就会有不同的行为或者心理. 这同样也是一种访问者模式的场景.

我们现在知道了访问者模式的应用场景了. 同样根据场景一来编写一下实现代码:

首先, 我们实现一个处方接口, 里面包括药物列表 以及 价格

```java
public interface Prescription {
    String getMadicineList();
    Double getPrice();
    void accept(Visitor visitor);
}
``` 

实现处方类具体类

```java
public class PrescriptionImpl implements Prescription {
    public String getMadicineList() {
        return "头孢、万通筋骨贴";
    }

    public Double getPrice() {
        return 128.0;
    }

    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
}
```

后面, 我们定义访问者接口

```java
public interface Visitor {
    void visit(Prescription prescription);
}
```

定义交费处访问者类

```java

public class PayCounterVisitor implements Visitor {
    public void visit(Prescription prescription) {
        System.out.println("扣除你社保卡上面" + prescription.getPrice() + "块钱");
    }
}

```

定义药房访问者类

```java
public class PharmacyVisitor implements Visitor {
    public void visit(Prescription prescription) {
        System.out.println("已为你准备好一下药物" + prescription.getMadicineList() + ", 请清点一下");
    }
}
```

这样我们就定义完成了所有的类, 那么我们客户端怎么调用呢?

```java
public class Client {
    public static void main(String[] args) {
        Prescription prescription = new PrescriptionImpl();
        Visitor pharmacyVisitor = new PharmacyVisitor();
        Visitor payCounterVisitor = new PayCounterVisitor();

        prescription.accept(payCounterVisitor);
        prescription.accept(pharmacyVisitor);
    }
}
```

有人可能会说, 上面类图中还有一个ElementContainer类还没有使用呢, 这个Container等到什么时候得的病比较重的时候, 医生一下给你开了2张以上处方的时候就需要用到了.嘿嘿~祝愿大家都不会用到该类.

### 实践心得

访问者模式在访问者这边具有扩展能力, 在被访问者这边也具有扩展能力. 比如当你再次去拿药的时候, 是另外一个医生接诊的. 那么你可以把你的上一次的处方给这位医生看, 他可以更好的了解你上一次的病情. 那么就需要增加一个医生访问者, 可以根据处方单来对你当前的病症做出一些辅助性的判断. 

而在被访问者也可以很容易的扩展, 同时与已经定义好的访问者配合使用. 比如现在除了医生开的处方, 我们还需要拍个CT, 那么CT的单子跟处方的单子都需要与收费处进行协作。但是CT不需要与药房Visitor配合使用, 但可以与医生访问者配合使用 以及 CT拍片室(需要扩展)配合使用.

虽然访问者模式具有扩展开放的特点, 但同时我们根据上面的代码也会发现, 该模式并不符合迪米特法则(最小知识原则). 因为收费处并不需要知道你开的药方是什么, 同时药房也并不需要知道你买的药花了多少钱.

注: 虽然上面举例说明了访问者模式应该如何使用, 但是在项目中确实没有使用到该模式. 等到什么时候读到更多的源码的时候再进行补充吧.

如有不同意见, 欢迎拍砖~