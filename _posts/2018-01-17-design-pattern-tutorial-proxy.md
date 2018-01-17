---
layout: post
title: 设计模式系列之代理模式
date: 2018-01-17 20:45:00
tags: [设计准则, 代理模式]
---

### 代理模式

代理模式: 为其他对象提供一种代理以控制对这个对象的访问。代理模式的出现第一是为了屏蔽客户端对于真实对象的调用,同时也能够通过代理对象为真实对象增加本不应该在真实对象中实现的一些工作. 做到对象的单一责任.

代理模式的通用类图:

![代理模式通用类图](/assets/images/2018-01-17-design-pattern-proxy.png)

类图中包含三个主要类:

1、Subject类: 主要定义抽象类的行为.

2、RealSubject: 负责实现Subject中定义的行为。

3、Proxy: 实现Subject抽象类或接口, 并且代理RealSubject类的行为. 使得调用方与真实对象之间有一个中间层. 完成除了RealSubject行为之外的其他一些行为.

### 应用场景

代理模式目前来看, 并没有在哪里有用到. 但是如果大家使用面向切面编程的话. 会在不知不觉中使用到动态代理模式. 通过切面、切入点的定义, 自动实现代理类, 在不怎么侵入具体对象实现的情况下, 增加了该具体类的一些其他额外的功能.

但是为了更好的说明代理类的使用, 这一部分就不对面向切面编程做更多的说明了. 后面会单独写一篇博客来介绍面向切面编程的实现细节与应用场景. 我们以演员出场演综艺节目为例, 来说明代理模式如何使用.

王宝强, 大家都不陌生. 宋喆就是王宝强的代理类. 现在各个综艺节目都知道如果要找明星上自己的节目都需要跟经纪人(代理类)沟通, 由经纪人安排行程、谈出场费、定飞机、收尾款. 而王宝强只需要上台表演就可以了. 在实现之前, 我们考虑一下如果没有代理类, 那么宝强如果要去某一个综艺节目需要怎么做呢?

1、电视台与宝强沟通上节目的事情

2、宝强要列出来最近一年的日程, 并且查看是否有时间去参加综艺节目

3、宝强需要跟节目组谈出场费

4、需要去携程上面定机票与酒店

5、需要表演, 表演完了之后还要跟踪尾款是否到账. 

我了个去, 宝强可是个大明星啊. 怎么可能搞定这么多事情呢? 那么这时候, 宋喆出现了. 对宝强说: "强哥, 你是演员, 你只需要表演就可以了, 其他工作就交给我了", 心里却想着: "照顾嫂子之类的工作就交给我了.".

好, 我们既然已经清楚真实场景了. 那么我们基于该场景进行编程.

首先开发一个Actor的类. 该类定义一个act的接口:

```java
public interface Actor {
    void act();
}
```

定义演员的实现类:

```java
public class BaoQiang implements Actor {
    public void act() {
        System.out.println("本宝宝要开始表演了.");
    }
}
```

定义经纪人类:

```java
public class SongZhe implements Actor {

    Actor actor;

    SongZhe(Actor actor) {
        this.actor = actor;
    }

    public void act() {
        System.out.println("安排行程");
        System.out.println("订酒店");
        System.out.println("订机票");
        System.out.println("谈出场费");
        this.actor.act();
        System.out.println("收尾款");
    }
}

```

客户端如何调用呢?

```java
public class Client {
    public static void main(String[] args) {
        Actor baoqiang = new BaoQiang();
        Actor proxy = new SongZhe(baoqiang);
        proxy.act();
    }
}
```

我们运行程序可以看到如下输出:

    安排行程
    订酒店
    订机票
    谈出场费
    本宝宝要开始表演了.
    收尾款

宝强除了表演, 其他的都被宋喆干了. 当然上面这个模式属于普通代理模式. 还有一种就是强制代理模式. 因为现在演员-经纪人都已经形成一种大家都认可的模式了. 如果在前些年可能如果有要好的朋友可能直接略过经纪人找到王宝强. 但是王宝强为了养马蓉, 还是需要听从经纪人的安排， 毕竟擅自答应了可能就会面临违约或者失信于朋友的风险. 当朋友要约宝强的时间的时候, 宝强会跟朋友说, 你去找我的经纪人, 看看我有没有时间。 这要怎么实现呢?

```java
public interface Actor {
    void schedule();
}

public class BaoQiang implements Actor {

    private Actor proxy;

    public void setProxy(Actor proxy) {
        this.proxy = proxy;
    }

    public void schedule() {
        if (proxy != null) {
            proxy.schedule();
        }
    }
}

public class SongZhe implements Actor {
    public void schedule() {
        System.out.println("最近宝宝没有时间, 改日再约");
    }
}
```

客户端如何调用呢?

```java
public class Client {
    public static void main(String[] args) {
        BaoQiang baoqiang = new BaoQiang();
        Actor proxy = new SongZhe();
        baoqiang.setProxy(proxy);
        baoqiang.schedule();
    }
}
```

对于请求到宝强这里的, 宝强就会强制把请求路由到代理对象上面. 这个就是强制代理模式. 但是这两种类型都不常见. 

有段时间我一直想不明白一件事, 就是可以通过装饰者模式来实现真实对象的扩展, 而不需要装饰者实现Subject接口, 这样更容易实现解耦与功能扩展 与 单一责任原则。

解耦表现在 装饰者类并不需要实现 被装饰类的接口. 而代理类却需要实现被代理类的接口.

功能扩展表现在: 装饰者不仅可以通过聚合的方式完成真实对象的调用, 而且可以增加自己的行为. 因为装饰者有自己的行为定义的接口. 而代理模式如果是面向接口编程, 那么只能具有被代理类对象实现的接口中的行为.

单一责任: 这个就更好理解了. 因为我们在上面定义的是一个Actor接口, 而代理类需要实现Actor接口, 本来应该是表演行为的实现中平白无故增加了很多与表演不相关的方法. 违反了单一责任原则. 

那么是不是说装饰者模式就比代理模式更好呢？ 在我认为, 至少在普通代理模式 与 强制代理模式这两种实现上, 使用装饰者模式更好。 但同时我们也有 动态代理模式, 动态代理模式因为是传过去的一个接口, 如果想不动声色的完成被代理对象的扩展的话. 使用代理模式才是最好的选择. 

### 实践心得

代理模式目前广泛应用在动态代理领域. 框架通过使用动态代理模式实现了AOP编程, 让系统的扩展更加容易. 同时Spring中定义了很多模板. 比如JPA中的JPARepository, 我们只需要继承该接口就可以通过接口中定义的方法操作数据库而不需要具体实现就是依托于动态代理完成的.