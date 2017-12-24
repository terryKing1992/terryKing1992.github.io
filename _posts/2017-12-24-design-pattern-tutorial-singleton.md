---
layout: post
title: 设计模式系列之开闭原则
date: 2017-12-24 20:49:00
tags: [设计模式, 单例模式]
---

### 单例设计模式

单例模式: Singleton Pattern; 是Java中最为常见也是最简单的一种设计模式, 为了确保在应用的生命周期范围内, 该类的实例永远只会出现一次(对于不会出现的情况, 那么该类也没必要存在)。 通过将构造函数私有化来完成

单例模式通用的类图:

![单例模式通用类图](/assets/images/2017-12-24-design-pattern-singleton-class-graph.png)

### 单例模式使用场景

在一个系统中, 要求一个类有且仅有一个对象, 如果出现多个对象会达不到系统需求; 
1、在整个项目中需要共享访问或者共享数据: 比如iOS中UIApplication类, 就需要保存当前应用的一些状态, 而这些状态在全局都是可以共享的.
2、在项目中对于一些类的创建性能比较低, 而且频繁会使用到. 比如数据库连接池, 存储数据库连接的多个实例. 但是数据库连接池一般设计为单例了. 在程序中任意地方调用都可以从同一个数据库连接池中获取数据库连接对象.
3、当需要定义一些静态方法的时候, 也可以使用单例模式(当然也可以使用静态类中的静态方法)

### 单例模式的实现

#### 饿汉式实现

饿汉式实现也是比较简单的实现方式, 即在类被加载的时候就通过final 类型的静态成员变量进行声明与创建.同时也可以避免由于多线程访问而造成线程安全问题;

    public class Singleton {
        private final static Singleton INSTANCE = new Singleton();

        private Singleton() {

        }

        public static Singleton getInstance() {
            return INSTANCE;
        }

        public void doSomething() {
            
        }
    }

当然我们也可以考虑Effective Java中提到的使用枚举类型来完成单例的实例化;

    public enum Singleton {
        INSTANCE();
        
        private Singleton() {
            
        }
        
        public void doSomething() {
            
        }
    }

#### 懒汉式实现

懒汉式实现是指在加载类的时候不会实例化单例类的实例, 而是等什么时候调用之后, 才会实例化出来该类的单例。我们都知道为了完成全局唯一的要求, 我们需要先将构造函数私有化, 同时提供获取实例的方法, 那么我们开始编码:

    public class Singleton {
        private static Singleton INSTANCE = null;

        private Singleton() {

        }

        public static Singleton getInstance() {
            if (INSTANCE == null) { //①
                INSTANCE = new Singleton();
            }
            return INSTANCE;
        }

        public void doSomething() {

        }
    }

上面实现的单例在单线程模式下可以运行良好, 但是在多线程环境下会出现线程安全问题. 由于线程的并发访问而造成出现多个实例的情况. 比如线程A 和 线程B 假设线程A先调用getInstance方法, 当线程A走到①所在行后, 由于A的CPU时间片结束, CPU转而执行线程B的getInstance方法, 当线程B走到①位置时发现 INSTANCE 确实是null的, 那么线程B会创建一个对象, 而当由线程A 执行的时候, 因为①语句已经执行过, 所以线程A继续往下执行也会创建出与B不一样的对象, 而造成单例模式的破坏. 

又由于当前的CPU环境全部都是多CPU或者多核的计算机组成, 那么我们就需要为防止线程安全的事情发生而使用锁机制避免因为多线程访问造成的线程安全问题. 下面我们修改程序, 使用锁来避免线程安全问题;

    public class Singleton {
        private static Singleton INSTANCE = null;

        private Singleton() {

        }

        public synchronized static Singleton getInstance() {
            if (INSTANCE == null) {
                INSTANCE = new Singleton();
            }
            return INSTANCE;
        }

        public void doSomething() {

        }
    }

这样我们完全避免了线程安全问题, 但同时也会因为在方法上加锁, 造成了线程在同时调用方法的时候会出现排队情况, 但其实方法内部也是可以存在一些并行的操作的, 这样做反而降低了程序的性能, 我们可以考虑在方法内部的部分语句上面加锁, 而不是在整个方法上面加锁;

    public class Singleton {
        private static Singleton instance = null;

        private Singleton() {

        }

        public static Singleton getInstance() {
            if (instance == null) {
                synchronized (Singleton.class) {
                    if (instance == null) {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }

        public void doSomething() {

        }
    }

这样做看着懒汉模式的实现已经接近完美了, 但是由于JVM在编译程序期间会对程序进行指令优化, 达到最终的一致性 而不是调用顺序与代码的调用顺序完全一致, 同样也会出现线程安全问题, 由于指令优化与重排而造成的线程安全问题会单独在写一篇文章用于记录. 我们又不得不面对重新修改程序的命运. 这次我们使用静态内部类, 既能够实现懒加载同时又可以完美的避免线程安全问题;

    public class Singleton {

        private final static class SingletonHolder {
            private final static Singleton INSTANCE = new Singleton();
        }

        private Singleton() {

        }

        public static Singleton getInstance() {
            return SingletonHolder.INSTANCE;
        }

        public void doSomething() {

        }
    }

到这里我们就可以看到单例模式在Java中的实现, 以及对于各个实现方式的比较;


### 单例设计模式心得

单例设计模式是设计模式中最为简单的一种设计模式, 应用也十分广泛; 如在Spring中, 由于使用了依赖注入的实现, 所有的Bean在默认情况下都是一个实例, 但不是说我们通过New的方式没法创建另外一个实例出来, 只是有我们自己创建的Bean实例不归Spring容器进行管理, 只能有JVM进行管理与回收. 一般来说, 在Spring项目中, 能用依赖注入的, 基本上不要自己创建类的实例;

