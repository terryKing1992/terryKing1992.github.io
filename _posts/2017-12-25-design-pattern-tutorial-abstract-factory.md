---
layout: post
title: 设计模式系列之抽象工厂模式
date: 2017-12-25 20:26:00
tags: [设计准则, 抽象工厂]
---

### 抽象工厂模式

抽象工厂模式一般用在多个产品下面有多个子产品的情况, 我们如果想要对该场景下的产品进行扩展, 那么抽象工厂模式就是最佳的代码设计思路了.(有些博客或者书籍中使用产品族与产品等级的概念容易让人混淆, 这里的产品与子产品可能表述与理解更加清晰)。抽象工厂模式的类图如下:

![抽象工厂模式类图](/assets/images/2017-12-25-design-pattern-abstract-factory.png)

### 应用场景

我们先看一个比较简单容易理解的场景: 假设现在我们需要产奥迪、奔驰的SUV 与 轿车, 当然SUV 与 轿车都具有代步的功能; 但是在舒适度方面SUV要比轿车要好很多; 那么我们现在开始基于该需求进行编程:

首先创建一个汽车的接口, 里面有代步功能 以及 舒适度访问:

    <% highlight java %>
    interface ICar {
        void run();
        int comfortDegree();
    }
    <% endhighlight %>

然后我们创建宝马车的抽象实现:

    <% highlight java %>
    abstract class AbstractBMWCar implements ICar {
        public void run() {
            System.out.println("宝马车开始跑了");
        }

        public abstract int comfortDegree();
    }
    <% endhighlight %>

同时创建奔驰车的抽象实现:

    <% highlight java %>
    abstract class AbstractBenzCar implements ICar {
        public void run() {
            System.out.println("奔驰车开始跑了");
        }

        public abstract int comfortDegree();
    }
    <% endhighlight %>

在分别实现SUV类型的车 与 普通轿车类之前, 我们先定义舒适度的指标:

    <% highlight java %>
    enum ComfortDegreeEnum {
        HIGH(10), MIDDLE(5), LOW(1);
        private int comfortDegree;

        ComfortDegreeEnum(int comfortDegree) {
            this.comfortDegree = comfortDegree;
        }

        public int getComfortDegree() {
            return comfortDegree;
        }

        public void setComfortDegree(int comfortDegree) {
            this.comfortDegree = comfortDegree;
        }
    }
    <% endhighlight %>

下面我们分别实现宝马的SUV、轿车与奔驰的SUV 与轿车:

    <% highlight java %>
    class BMWSuvCar extends AbstractBMWCar {
        public int comfortDegree() {
            return ComfortDegreeEnum.HIGH.getComfortDegree();
        }
    }

    class BMWSaloonCar extends AbstractBMWCar {
        public int comfortDegree() {
            return ComfortDegreeEnum.LOW.getComfortDegree();
        }
    }

    class BenzSuvCar extends AbstractBenzCar {
        public int comfortDegree() {
            return ComfortDegreeEnum.HIGH.getComfortDegree();
        }
    }

    class BenzSaloonCar extends AbstractBenzCar {
        public int comfortDegree() {
            return ComfortDegreeEnum.MIDDLE.getComfortDegree();
        }
    }
    <% endhighlight %>

既然产品已经全部就位了, 我们需要开始构建工厂类的模型了

首先, 经过分析我们发现, 对于宝马或者奔驰下面都需要创建2中不同类型的车; 我们先定义工厂接口类

    <% highlight java %>
    interface CarFactory {
        ICar createSuvCar();
        ICar createSaloonCar();
    }
    <% endhighlight %>

同时, 分别实现宝马与奔驰的工厂类:

    <% highlight java %>
    class BMWCarFactory implements CarFactory {
        public ICar createSuvCar() {
            return new BMWSuvCar();
        }

        public ICar createSaloonCar() {
            return new BMWSaloonCar();
        }
    }

    class BenzCarFactory implements CarFactory {
        public ICar createSuvCar() {
            return new BenzSuvCar();
        }

        public ICar createSaloonCar() {
            return new BenzSaloonCar();
        }
    } 
    <% endhighlight %>

这样我们基本上就实现了创建产品下子产品的建模过程. 因为我们设计的程序需要对扩展开放, 对修改关闭; 同时接口是类行为的契约, 我们明白改变契约所带来的影响是很大的. 所以我们不能当一需要增加需求的时候就改接口, 那样必然会带来不可预知的问题. 所以, 在上述程序中, 增加CarFactory的方法是不可取的. 那么我们就很容易记清楚, 对扩展开放是对什么扩展开放了, 既然子产品的种类没法增加, 那么必然是可以增加产品的种类的.

比如, 如果我们需要增加一个Aud类型的SUV 与 普通轿车, 那么我们只需要增加 Audi的抽象类, 同时增加AudiSUV 以及 AudiSaloonCar的类, 然后增加Audi的工厂类即可完成Audi两种类型的车的创建, 对扩展完全开放. 只需要增加类就可以完成对功能的扩展, 而客户端也只需要修改少量的代码就可以完成切换. 而且客户端代码基本上属于上层代码, 如果需求变动的话, 上层代码不动基本上也是不可能的.

通过抽象工厂模式我们做到了很大程度上的对扩展开放对修改关闭;

### 实践心得

在开发过程中, 对于某一种设计模式不太了解的情况下, 很难做到在某一些场景下用哪个设计模式, 据我的经验来看当开发的产品下面有多个子产品需要创建, 并且父产品也不止有一种的情况下, 就可以使用抽象工厂模式; 比如富士康工厂中代加工电子产品, 我们知道电子产品除了苹果公司的, 还有小米公司的都会找富士康代加工; 那么苹果公司产品 与 小米公司产品就属于父产品级别, 而 苹果公司下面的iPhone、iPad属于子产品, 小米公司同样有phone 与 pad的业务。这样我们在定义富士康工厂模型的时候, 就可以任务富士康里面有两个车间(苹果车间、小米车间), 分别可以从车间中获得手机产品与Pad产品; 如果什么时候华为、Oppo等手机厂商也去找富士康代加工, 那么只需要额外增加两个车间就可以了, 并不影响苹果与小米的生产;

