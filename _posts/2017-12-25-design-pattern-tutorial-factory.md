---
layout: post
title: 设计模式系列之工厂模式
date: 2017-12-25 10:20:00
tags: [设计模式, 工厂模式]
---

### 工厂模式

工厂模式是为了将创建对象 与 对象的实现分离, 从而能够更好的解耦客户端对于具体对象的依赖, 转而只依赖对象的抽象 以及 具体的工厂. 
从而达到低耦合高内聚 以及 易扩展的特性;工厂模式分为3种: 简单工厂、工厂方法 以及 抽象工厂; 
而本片博客主要记录创建一个产品族下面的多个产品的的应用场景: 简单工厂 和 工厂方法;

### 简单工厂

简单工厂, 作为经常使用的一种设计模式, 能够提供客户端 与 具体对象的依赖; 我们可以想象一下如果一个司机不知道如何造车, 那么他能否开车呢? 同样的道理, 对于客户端即便不知道具体对象如何创建, 他也可以委托创建工厂帮他创建, 自己只要会用就可以了. 这样对于更加细化的分工更加利于扩展与维护;

简单工厂的类图如下:

![简单工厂类图](/assets/images/2017-12-25-design-pattern-simple-factory.png)

### 应用场景

下面我们看一个应用场景, 就是上面汽车的例子, 我们现在有个客户需要使用车, 第一种方式就是客户自己知道如何创建车, 并且自己试用车:

```java
    public class Client {
        public static void main(String[] args) {
            Car audiCar = new AudiCar();
            Car bmwCar = new BMWCar();

            audiCar.run();
            bmwCar.run();
        }
    }

    interface Car {
        void run();
    }

    class AudiCar implements Car {
        public void run() {
            System.out.println("奥迪车启动, 要开始跑了");
        }
    }

    class BMWCar implements Car {
        public void run() {
            System.out.println("宝马车启动, 要开始跑了");
        }
    }
```

上面实现的代码基本上可以满足需求了, 但是问题来了, 司机需要知道宝马 奥迪车如何创建, 如果新增一个汽车种类, 那么司机也要知道怎么创建. 相当于司机要拥有N 项技能才能随意的开自己想开的车; 同时司机与具体的车具有了耦合情况, 其实司机如果拿到驾驶证, 什么样的车都可以开; 司机不需要与车的具体的实现类耦合; 如果想要松耦合, 我们可以引入简单工厂模式;

```java
    public class Client {
        public static void main(String[] args) {
            String carType = "audi";
            Car car = SimpleFactory.createCar(carType);
            car.run();
        }
    }

    class SimpleFactory {
        public static Car createCar(String carType) {

            if ("audi".equals(carType)) {
                return new AudiCar();
            }else if ("bmw".equals(carType)) {
                return new BMWCar();
            }

            return null;
        }
    }

    interface Car {
        void run();
    }

    class AudiCar implements Car {
        public void run() {
            System.out.println("奥迪车启动, 要开始跑了");
        }
    }

    class BMWCar implements Car {
        public void run() {
            System.out.println("宝马车启动, 要开始跑了");
        }
    }
```

引入SimpleFactory类, 根据司机传入的车的名称来创建具体的车的对象, 我们通过增加类减少了客户端与具体实现类的耦合, 有人可能有这样的疑问: 虽然减少了对具体实现类的依赖, 但是引入了新的依赖呀? 是的, 我们是引入了新的依赖, 但是客户端对于新的依赖, 只有一种依赖就是SimpleFactory, 而客户端对于AudiCar、BMWCar等依赖就有了好多依赖了. 还有人可能存在另外一种疑问, 那SimpleFactory 也需要对各个厂家的车具有依赖关系, 在系统中并没有减少依赖, 反而增加了一个中间类, 导致系统的依赖关系更加复杂了? 真的是这样吗? 确实是这样, 整个系统的依赖关系变的复杂了, 但是我们要从另外一个角度去看问题, 毕竟我们编写的代码都是为了上层服务的, "以客户为中心", 那么我们这么做极大的增加了客户的便利性;

而在编程世界中, 能够尽量减少因为需求变动引发的整个系统上下层的变动, 才是设计模式所带来的优势. 有时候确实需要上层的变动, 但我们也要尽可能的让上层变动的越少越好, 变动越少说明系统的扩展性越高、由于修改代码造成的不可预测的问题的可能性越小;

但同时我们观察SimpleFactory也很容易发现, 该类对于基本上是对于扩展关闭, 对于修改开放; 怎么理解这一句话呢? 因为如果我们需要增加奔驰车, 那么我们就需要修改createCar这个方法, 而且改动If else代码, 这个风险相对来说还是很大的; 既然这样, 我们是不是可以找到一个更好的简单工厂的实现呢?

当然, 我们为了更好的减少对于某一个方法的变动, 而增加方法对客户端影响会相对较小一些, 我们可以将原来的createCar方法拆分为2个方法甚至更多:

```java
    public class Client {
        public static void main(String[] args) {
            Car car = SimpleFactory.createAudiCar();
            car.run();
        }
    }

    class SimpleFactory {
        public static Car createAudiCar() {
            return new AudiCar();
        }

        public static Car createBMWCar() {
            return new BMWCar();
        }
    }
```

这样来看的话, 如果我们需要增加一个奔驰车, 那么我们只需要创建一个类, 并且给SimpleFactory增加一个新的方法即可. 在项目开发过程中, 一般依据我的经验来说的话, 对于需求的变动, 我们应该尽量设计的代码能够体现如下原则:

    1、能够通过增加类就完成的, 一定要通过增加类来完成
    2、如果能够通过增加类的方法完成的功能, 一定要通过增加类的方法完成
    3、实在没办法的时候, 首先要看一下自己的代码结构是否合理, 如不合理则需要重构
    4、 如果相对来说还比较合理,那么一定要做好该修改模块的单元测试.把底层安全网搭建起来

当然, 简单工厂目前只能够做到通过增加类的方法来完成功能的扩展. 下面我们看一个简单工厂的升级版, 工厂方法模式.

### 工厂方法模式

工厂方法模式与简单工厂模式运用的场景基本上一模一样, 但是工厂方法模式比简单工厂模式具有更好的扩展性 与 面向接口编程的特性;
下面我们看一下, 对于上面提到的同一个问题, 应该如何通过工厂模式更加优雅的完成系统的开发.

工厂方法类图如下:

![工厂方法模式类图](/assets/images/2017-12-25-design-pattern-factory-method.png)

首先, 我们需要创建我们的产品类, 这个代码是不变的.

```java
    interface Car {
        void run();
    }

    class AudiCar implements Car {
        public void run() {
            System.out.println("奥迪车启动, 要开始跑了");
        }
    }

    class BMWCar implements Car {
        public void run() {
            System.out.println("宝马车启动, 要开始跑了");
        }
    }
```

后面我们创建工厂的抽象接口:

```java
    interface CarFactory {
        Car createCar();
    }
```

然后创建奥迪 与 宝马的工厂具体实现

```java
    class BMWFactory implements CarFactory {
        public Car createCar() {
            return new BMWCar();
        }
    }

    class AudiFactory implements CarFactory {
        public Car createCar() {
            return new AudiCar();
        }
    }
```

那么客户端怎么调用呢?

```java
    public class Client {
        public static void main(String[] args) {
            CarFactory carFactory = new BMWFactory();
            Car car = carFactory.createCar();
            car.run();
        }
    }
```

那么, 如果想增加一个奔驰车怎么办呢? 我们只需要增加一个奔驰车类 以及 一个奔驰车对应的工厂类即可, 并且根据依赖倒置原则, 客户端依赖了工厂的抽象, 而不是依赖的具体实现; 工厂方法同时提供了高扩展性. 

### 实践总结

工厂方法模式在项目中使用的非常频繁, 只要项目中有同一个产品族下面的多个产品需要创建的时候, 基本上都会用到工厂模式. 而且基本上工厂模式也会跟其他的设计模式一块使用, 比如模板方法等等;

工厂模式 与 模板方法在 我以前项目中使用的场景需求

    1、为了完成扩展, 定义了3中消息队列的实现, RocketMQ、Kafka、 RabbitMQ
    2、如果要完成消息的发送与接收任务, 需要定义Consumer 与 Producer的抽象类
    3、而Consumer 与 Producer的具体类的构造方法基本上都是 指定端口, 指定IP, 以及不同的Topic
    4、在客户端使用的时候, 可以根据用户传入的消息队列类型来创建这些Consumer以及Producer对象

那么我们总结一下该需求中可能遇到什么设计模式; 针对第三条, 我们可以定义骨干类, 并且实现使用@PostConstruct注解完成一些参数的创建工作, 而 在骨干类中定义 getConsumerIP 以及 getConsumerPort、getTopic等抽象方法由子类实现; 我们用到了模板方法; 针对第四条, 根据传入参数的不不同, 创建选择不同的工厂, 创建不同的消息队列的消费者与生产者, 我们可以用到简单工厂或者工厂方法. 不过其实这边使用抽象工厂更加容易扩展, 毕竟是多个产品族( RocketMQ、Kafka、 RabbitMQ)下面的多个产品(Consumer、Producer)的创建。

上面提到的抽象工厂, 我们会在下一篇博客中继续学习;


