---
layout: post
title: 设计模式系列之接口隔离原则
date: 2017-12-20 19:49:00
tags: [设计准则, 接口隔离原则]
---

### 接口隔离原则

接口隔离原则的定义: 客户端不应该依赖它不需要的方法; 一个类对另一个类的依赖应该建立在最小接口上面;

### 场景示例

举一个简单的例子吧, 虽然不不懂车, 但是我看到豪车也觉得很帅. 这次我们就拿车作为例子进行说明; 假设我们现在有两种车, 一种是能够开盖的跑车, 一种是普通的轿车. 但是他们能够提供载人的功能;我们基于该场景开发一个程序; 首先定义 一个车行为的接口ICar, 同时实现两个具体的类 

```java
    public interface ICar {
        void run();
        void openRoof() throws UnsupportedOperationException;
    }

    class OpenRoofCar implements ICar {
        public void run() {
            System.out.println("我已经在全力跑了");
        }

        public void openRoof() throws UnsupportedOperationException {
            System.out.println("我还有更拉风的动作, 开启车顶来兜风");
        }
    }

    class NormalCar implements ICar {
        public void run() {
            System.out.println("疾驰的感觉就是爽");
        }

        public void openRoof() throws UnsupportedOperationException{
            throw new UnsupportedOperationException();
        }
    }
```

瞬间就完成了对于程序的开发, 但同时我们也发现了一些问题, 该程序并没有遵守接口隔离原则, 因为NormalCar依赖了他不需要的接口;openRoof接口. 下面为了使得程序遵守接口隔离原则, 我们对程序进行一些修改.首先将ICar接口分离成两个接口INormalCar 以及 IOpenRoofCar 这样的话, 我们的实现类就不会依赖不需要的接口了. 下面看修改后的代码:

INormalCar接口:

```java
    interface INormalCar {
        void run();
    }
```

IOpenRoofCar接口 继承 INormalCar接口:

```java
    interface IOpenRoofCar extends INormalCar {
        void openRoof();
    }
```

具体的实现类如下:

```java
    class OpenRoofCar implements IOpenRoofCar {
        public void run() {
            System.out.println("我已经在全力跑了");
        }

        public void openRoof() throws UnsupportedOperationException {
            System.out.println("我还有更拉风的动作, 开启车顶来兜风");
        }
    }

    class NormalNormalCar implements INormalCar {
        public void run() {
            System.out.println("疾驰的感觉就是爽");
        }
    }
```

这样我就就做到了接口隔离;

### 接口隔离原则的使用心得

个人理解, 不保证正确;

1、在定义接口的时候, 如果实现类中实现的接口的方法有些是不需要的, 那么尽快将接口再进行分割; 在目前的项目中就存在这样的问题
    项目中使用接口 + 抽象骨干类 + 具体实现的方式来定义两条大概相同的流水线, 但是在A流水线中并不需要D功能, 但是B流水线中需要D功能, 而整个模板方法中的流程一模一样, 只有在该 该功能的调用这一句话上面不一样, 且该功能处于流水线的中间环节. 目前的做法是在AbstractBackbone重定义了抽象方法, 即DFeature(), 同时让A流水线实现一个空的方法, B流水线实现D功能, 这样在在骨干方法中调用this.DFeature()方法即可, 但不可否认, 这样违背了接口隔离原则; 但是目前没有找到更好的办法来做到既能够最大程度的重用代码, 又符合接口隔离原则.

2、在使用接口隔离原则的时候, 同样要符合单一责任原则. 因为虽然有时候抽象出来的N个接口 子类必须都要实现, 这样确实符合接口隔离原则, 但是由于 有些接口是不属于同一类型的行为的, 我们也要使用单一责任原则为指导思想将接口分离;

3、因为接口隔离之后, 那么当我们使用接口作为引用去定义一个对象的时候 比如```INormalCar normalCar```; 的时候 要用到最小的需要的接口, 如果客户端只需要run这个功能, 我们在客户端定义引用的时候使用```IOpenRoofCar car``` 这样也是不符合接口隔离原则的. 因为我们只需要最小的需要使用的接口.

### 接口隔离原则 与 单一责任原则的区别

在以前某个时间段里, 我总觉得单一责任原则跟接口隔离原则都是在做同样的事情, 就是将接口尽量的拆分的细粒度一些. 但是认识还是比较片面, 其实单一责任原则 与 接口隔离原则起到了相辅相成的作用. 我们举个栗子就会明白了.

我们在单一责任原则的博客里面，我们提到了数据库相关的接口, 现在我们对他进行一点变形:

```java
    public interface IDBConnection {
        String getDBDriverName();
        String getDBUrl();
        String getDBUserName();
        String getDBPassword();
        Connection getDBConnection();
        String closeConnection();
        //去掉executeSQL接口
        <!-- Object executeSQL(String sql) -->
    }
```

我们可以看到, 因为基本上使用JDBC操作数据库, 这些数据是必须要定义的 即在数据库连接接口的实现类中以上定义的方法都是需要实现的, 接口中并没有实现类不需要的方法. 那我们可以说该接口的定义符合接口隔离原则. 但是经我们上次分析, 该接口定义并不符合单一责任原则; 同样的, 符合单一责任原则的 有时候 又不一定符合接口隔离原则, 所以基于该两项原则去开发接口才能够具有很好的效果.
