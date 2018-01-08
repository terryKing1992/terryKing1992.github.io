---
layout: post
title: 设计模式系列之状态模式
date: 2018-01-07 20:00:00
tags: [设计准则, 状态模式]
---

### 状态模式

当一个对象内在状态改变时允许其改变行为, 这个对象看起来像是改变了其类.状态模式的核心就是封装, 使用Context类来封装状态, 当Context类中的state对象发生改变时, 从外部看就好像这个对象对应的类发生了改变一样.

我们来看一下状态模式的通用类图:

![状态模式通用类图](/assets/images/2018-01-07-design-pattern-state.png)

类图中包括三个关键类:

1、State抽象类: 用于定义状态应该具有的行为, 以及一个protected 的Context类

2、ConcreteState: 用于实现State并且针对具体的状态实现不同的行为

3、Context类: 用于封装状态的调用, 并且在Context类的内部完成状态的切换

### 应用场景

因为在我们的编程世界中, 有很多事物都存在状态. 因为以前开发消息推送系统, 我们就以消息的状态为例来说明状态模式的使用. 一般来说我们认为推送的消息分为3种状态, ```初始状态``` ```推送中``` ```已送达```状态. 那么我们根据这三个状态来开发程序, 在开发程序之前, 我们先看一下这些状态的有限状态机是什么样子的?

![消息状态的有限状态机](/assets/images/2018-01-07-design-pattern-state-machine.png)

那么如果我们还是以最原始的冲动来编程, 可能我们最先想到的还是if else编程. 那么我们还是看一下如何来写代码?

```java
    public static void main(String[] args) {
        String initState = "初始化";
        String state = initState;
        while(true) {
            if ("初始化".equals(state)) {
                System.out.println("在数据库中插入一条消息, 消息的状态为" + state);
                state = "推送中";
            }else if("推送中".equals(state)) {
                System.out.println("将刚才插入数据库中的数据状态由初始化修改为" + state);
                state = "推送完成";
            }else {
                System.out.println("将刚才插入数据库中的数据状态由推送中修改为" + state);
                break;
            }
        }
    }
```

那么, 我们可以看到通过一个while循环的方式 + if判断的方式能够完成状态的切换, 并且对于不同的状态输出不同的打印日志; 还是前面经常提到的问题, 如何扩展呢? 并不符合开闭原则; 同时由于该类需要维护多个状态的行为、全部状态的切换工作. 也不符合单一责任原则; 那么当我们看到有些对象有多个状态, 并且状态的变化会引起行为的变化的时候, 我们可以考虑使用状态模式来进行代码的开发或者重构. 下面我们使用状态模式对以上需求进行重构:

1、首先, 我们需要创建状态类的抽象类或者接口; 基于接口优于抽象的原则, 我们使用接口来进行

```java
    interface PushMessageState {
        void handle();
        void setPushMessageContext(PushMessageContext pushMessageContext);
    }
```

该接口定义了状态应该具有的行为, 同时需要提供设置上下文环境的方法. 为后面在状态内部改变Context的状态做准备

2、定义PushMessageState的骨架实现

```java
    abstract class AbstractPushMessageState implements PushMessageState {

        PushMessageContext pushMessageContext;

        public void setPushMessageContext(PushMessageContext pushMessageContext) {
            this.pushMessageContext = pushMessageContext;
        }

        public abstract void handle();
    }
```

该骨架实现最主要的是实现上下文环境的设置, 用于后面在状态内部改变Context的状态做铺垫.

3、实现各个状态的行为实现

```java

    class InitPushMessageState extends AbstractPushMessageState {

        public void handle() {
            System.out.println("在数据库中插入一条消息, 消息的状态为" + this.getClass().getSimpleName());
            this.pushMessageContext.setPushMessageState(new InPushingMessageState());
        }
    }

    class InPushingMessageState extends AbstractPushMessageState {

        public void handle() {
            System.out.println("将刚才插入数据库中的数据状态由初始化修改为" + this.getClass().getSimpleName());
            this.pushMessageContext.setPushMessageState(new CompletedPushMessageState());
        }
    }

    class CompletedPushMessageState extends AbstractPushMessageState {

        public void handle() {
            System.out.println("将刚才插入数据库中的数据状态由推送中修改为" + this.getClass().getSimpleName());
        }
    }
```

步骤一的PushMessageState接口中我们定义的```setPushMessageState```起到了作用, 在状态内部改变上下文的状态, 并且对外提供不同的行为;

那么我们看一下客户端在怎么调用呢?

```java
    public class Client {
        public static void main(String[] args) {
            PushMessageContext pushMessageContext = new PushMessageContext();
            pushMessageContext.setPushMessageState(new InitPushMessageState());

            pushMessageContext.handle();
            pushMessageContext.handle();
            pushMessageContext.handle();
        }
    }
```

客户端只需要三次调用上层封装的行为即可. 并不需要知道状态之间到底应该怎么变化. 同时如果在创建PushMessageContext对象的时候, 就有一个默认的消息状态的实现, 那么客户端根本不需要知道都有哪几种状态就可以完成整个状态链的调用. 完全解耦了客户端调用与状态之间的耦合.  只需要依赖一个Context类即可. 符合高内聚低耦合的特点.

### 实践心得

状态模式适用于一个对象具有多个状态的场景下, 并且各个状态下的行为都是不一样的. 在开发过程中, 虽然上文中提到的是推送消息的状态的变化, 但是实际场景中并没有用到该设计模式, 毕竟这些状态都是异步实现的. 而在我们学习编译原理时候, 需要开发一个词法解析器的时候, 在实现有限状态机的时候就完全可以使用状态模式来完成开发. 根据当前所处的状态的不同, 遇到 ", ; () {} [] / " 等符号的时候需要做不同的处理;