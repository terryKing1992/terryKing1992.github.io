---
layout: post
title: 设计模式系列之中介模式
date: 2018-01-01 20:49:00
tags: [设计准则, 中介者模式]
---

### 中介模式

用一个中介对象封装一系列的对象交互, 中介者使各个对象不需要显式的互相调用, 从而使其松耦合, 而且可以独立的改变他们之间的交互.

中介者模式的核心思想就是将原来网状的调用关系进行梳理, 通过增加一个新的中介者对象, 来将原来的网状关系梳理成为星型关系. 从而降低各个同事类之间的耦合.

我们看一下具体的中介者模式的类图:

![中介者模式类图](/assets/images/2018-01-02-design-pattern-mediator.png)

### 中介者模式的应用场景

由于在实际的开发过程中目前还基本没有怎么用到中介者模式, 但是为了把该设计模式讲述清楚, 我们使用路由器的网络结构来进行说明. 假设现在有个需求, 现在有几台电脑, 电脑之间需要能够在局域网内进行通信. 我们知道, 网络拓扑结构有网状结构以及星型结构.

如果我们要使用网状结构来实现电脑之间的网络互联, 应该如何来实现呢? (我们假设一台电脑上面网卡是可以插任意多张的)

首先, 如果A电脑与B、C、D等电脑通信, 肯定需要知道B、C、D的局域网IP地址(同时为了演示同事类的继承, 我们假设B\C\D电脑都是不同牌子的). 那么我们就基于上面的需求进行编程; 

首先, 我们应该建立一个接口, 用来与不同的机器之间进行通信的接口(发送报文、接收报文):

    interface Computer {
        void sendMessage(String to, String content);
        void receiveMessage(String from, String content);
    }

后面分别实现A、B、C、D电脑的子类:


    abstract class AbstractComputer implements Computer {

        public void sendMessage(Computer to, String content) {

        }

        public void receiveMessage(Computer from, String content) {

        }
    }

    class A extends AbstractComputer {
    }

    class B extends AbstractComputer {
    }

    class C extends AbstractComputer {
    }

    class D extends AbstractComputer {
        
    }

当有了电脑之后, 我们要怎么才能让这些电脑之间进行通信呢?当然还需要在各个电脑中保持其他电脑的引用了:

    abstract class AbstractComputer implements Computer {

        private Map<String, Computer> colleaguesMap = new HashMap<String, Computer>();
        private String ip;

        AbstractComputer(String ip) {
            this.ip = ip;
        }

        public void register(String ip, Computer computer) {
            this.colleaguesMap.put(ip, computer);
        }

        public void sendMessage(String to, String content) {
            System.out.println("电脑" + this.getClass().getSimpleName() + "发送给电脑" + this.colleaguesMap.get(to).getClass().getSimpleName() + "的消息:" + content);
            this.colleaguesMap.get(to).receiveMessage(ip, content);
        }

        public void receiveMessage(String from, String content) {
            System.out.println("电脑" + this.getClass().getSimpleName() + "收到来自电脑" + this.colleaguesMap.get(from).getClass().getSimpleName() + "的消息:" + content);
        }

        public String getIp() {
            return ip;
        }

        public void setIp(String ip) {
            this.ip = ip;
        }
    }

我们需要给电脑类增加一个保持其他电脑的引用, 以及注册的方法.  到时候A电脑就可以与其他电脑通信了. 其他的电脑实现是一样的.

我们下面看一下客户端怎么调用呢?

    class MediatorDemo {
        public static void main(String[] args) {

            A computerA = new A("192.168.1.100");
            B computerB = new B("192.168.1.101");
            C computerC = new C("192.168.1.102");
            D computerD = new D("192.168.1.103");

            computerA.register(computerB.getIp(), computerB);
            computerA.register(computerC.getIp(), computerC);
            computerA.register(computerD.getIp(), computerD);

            computerB.register(computerA.getIp(), computerA);
            computerB.register(computerC.getIp(), computerC);
            computerB.register(computerC.getIp(), computerC);

            computerC.register(computerA.getIp(), computerA);
            computerC.register(computerB.getIp(), computerB);
            computerC.register(computerD.getIp(), computerD);

            computerD.register(computerA.getIp(), computerA);
            computerD.register(computerB.getIp(), computerB);
            computerD.register(computerC.getIp(), computerC);

            computerA.sendMessage("192.168.1.101", "你好啊");

            computerB.sendMessage("192.168.1.102", "你好啊");

        }
    }

如果按照上面的代码实现, 确实也能够完成各个电脑之间的通信功能, 只是代码看起来太长了, 而且扩展起来也非常的不方便, 最主要体现在客户端调用方面, 如果我现在需要增加一个电脑E跟各个电脑之间能够通信, 那么需要构造分别在多次调用A\B\C\D的register方法. 同时将A\B\C\D分别注册到E类中. 我的天啊, 这绝对不是人类能够看懂的代码了. 

而上面代码的实现类图如下所示:

![上面代码实现的类图](/assets/images/2018-01-02-design-pattern-net-shape-structure.png)

既然当很多类之间都存在通信的时候, 网状结构会造成这么多的问题, 我们使用星状结构来重新实现该需求. 看看能否满足扩展性与类间松耦合的原则呢？

当然, 需要增加中介者接口:

    interface Communication {
        void registerPeer(String ip, Computer computer);
        void sendMessage(String from, String to, String content);
        void receiveMessage(String from, String to, String content);
    }

同时实现具体的中介者Router:

    class Router implements Communication {

        private Map<String, Computer> colleaguesMap = new HashMap<String, Computer>();

        public void registerPeer(String ip, Computer computer) {
            colleaguesMap.put(ip, computer);
        }

        public void sendMessage(String from, String to, String content) {
            Computer computerFrom = this.colleaguesMap.get(from);
            Computer computerTo = this.colleaguesMap.get(to);
            System.out.println("电脑" + computerFrom.getClass().getSimpleName() + "发送消息给电脑" + computerTo.getClass().getSimpleName() + ", 消息为:" + content);
        }

        public void receiveMessage(String from, String to, String content) {
            Computer computerFrom = this.colleaguesMap.get(from);
            Computer computerTo = this.colleaguesMap.get(to);
            System.out.println("电脑" + computerTo.getClass().getSimpleName() + "收到来自电脑" + computerFrom.getClass().getSimpleName() + "的消息:" + content);
        }
    }

同时我们在Computer的子类中, 增加对于Router这个中介者的引用, 并且把发送消息与接收消息的功能交给中介者去完成;

    interface Computer {
        void sendMessage(String to, String content);
        void receiveMessage(String from, String content);
    }

    abstract class AbstractComputer implements Computer {
        private String ip;
        private Communication communication;

        AbstractComputer(String ip, Communication communication) {
            this.ip = ip;
            communication.registerPeer(ip, this);
            this.communication = communication;
        }

        public void sendMessage(String to, String content) {
            this.communication.sendMessage(this.ip, to, content);
        }

        public void receiveMessage(String from, String content) {
            this.communication.receiveMessage(from, this.ip, content);

        }

        public String getIp() {
            return ip;
        }

        public void setIp(String ip) {
            this.ip = ip;
        }
    }

    class A extends AbstractComputer {
        public A(String ip, Communication communication) {
            super(ip, communication);
        }
    }

    class B extends AbstractComputer {
        public B(String ip, Communication communication) {
            super(ip, communication);
        }
    }

    class C extends AbstractComputer {
        public C(String ip, Communication communication) {
            super(ip, communication);
        }
    }

    class D extends AbstractComputer {
        public D(String ip, Communication communication) {
            super(ip, communication);
        }
    }

这样我们就将原来的网状结构变为了星状结构, 下面我们看一下客户端调用:

    class MediatorDemo {
        public static void main(String[] args) {
            Communication router = new Router();
            A computerA = new A("192.168.1.100", router);
            B computerB = new B("192.168.1.101", router);
            C computerC = new C("192.168.1.102", router);
            D computerD = new D("192.168.1.103", router);

            computerA.sendMessage("192.168.1.101", "你好啊");
            computerB.sendMessage("192.168.1.102", "你好啊");
        }
    }

这么看来, 切实比原来简单好多. 如果要增加另外一个电脑的话, 我们只需要创建一个子类E, 并且客户端直接创建出来一个E然后就可以跟任意的电脑通信了. 中介者模式在这种场景下既减少了客户端调用的代码, 同时也增加了同事类的扩展性.

### 实践心得

在类图中出现蜘蛛网装结构, 一定要考虑使用中介者模式, 既能够提供系统的扩展性, 又能够使得调用逻辑清晰简单.




