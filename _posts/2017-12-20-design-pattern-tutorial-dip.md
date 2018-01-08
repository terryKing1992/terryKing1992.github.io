---
layout: post
title: 设计模式系列之依赖倒置原则
date: 2017-12-20 19:49:00
tags: [设计准则, 依赖倒置原则]
---

### 依赖倒置原则

依赖倒置的定义如下: 1、高层模块不应该依赖底层模块(这里指底层模块的具体实现); 2、高层模块与低层模块都应该依赖其抽象 3、抽象不应该依赖细节 而细节应该依赖抽象;
编程其实就是对实体或者概念的一种抽象, 而抽象主要是为了能够简单的描述物体具有的特性 与 行为, 抽象是从多个共同的物体上面抽取相同特征集合的过程, 所以抽象出来的概念具有普适性 与 通用性, 而且相对细节来说不会发生太多的变化。如果我们在上层模块中依赖抽象, 那么也同样意味着我们上层模块的变动概率会大大减小. 但如果上层模块依赖 具体实现的话, 那么由于 具象化物体 具有多样性, 上层模块依赖具体实现会造成 上层模块的代码频繁变动导致系统不稳定、难扩展的囧境。

### 应用举例

在2016年-2017年间, 最火的莫过于房地产了; 对于炒房客与刚需客在买房的时候都需要找代理了解房子的情况; 那么我们就基于该场景写一下代码, 首先我们知道房产中介主要的功能就是: 1、帮客户找优质房源 2、带领客户看房 3、签购房合同。 那么我作为一个客户要去买房了, 但是房产中介我只知道链家; 那么我们开发一个基于链家房产中介的买房程序; 总共有3个类, 买房客Client, 房产中介: LianJiaEstateAgent 以及 中介提供商 Demo

```java
    public class Demo {

        public static void main(String[] args) throws Exception {
            Client client = new Client();
            client.setAgent(new LianJiaEstateAgent());
            client.findBetterHouse();
        }
    }

    public class Client {
        private LianJiaEstateAgent  agent;
        public void setAgent(LianJiaEstateAgent agent) {
            this.agent = agent;
        }

        public void findBetterHouse() {
            this.agent.findBetterHouse();
        }

        public void visitHouse() {
            this.agent.visitHouse();
        }

        public void signContract() {
            this.agent.signContract();
        }
    }

    public class LianJiaEstateAgent {
        public void findBetterHouse() {
            System.out.println("客官您好, 我们已经帮您找到了优质房源");
        }

        public void visitHouse() {
            System.out.println("客官, 我们现在带您参观一下房子");
        }

        public void signContract() {
            System.out.println("客官, 您满意我们现在就签购房合同");
        }
    }
```

我们找到链家, 发现链家确实能够提供给我想要的服务; 但是我因为钱不够, 买不起链家提供的房源;但是因为现在我只认链家这一个中介, 其他的我不信任, 所以注定这辈子娶不到媳妇儿, 但是通过中介提供商Demo 对我的一些开导 以及 自己上网查了一下关于房天下这个中介的信誉度等等综合评价, 我发现这个房天下中介也不错, 我终于接受了房天下这个房产中介. 自此, 我接受了2个中介; 那么反应在程序中就是修改Client的代码, 增加一个```setFTXAgent()``` 方法, 并创建一个房天下的类FTXEstateAgent;

```java
    public static class Client {
        private LianJiaEstateAgent  agent;
        private FTXEstateAgent ftxEstateAgent;

        public void setAgent(LianJiaEstateAgent agent) {
            this.agent = agent;
        }

        public void setFTXAgent(FTXEstateAgent agent) {
            this.ftxEstateAgent = agent;
        }

        public void findBetterHouse() {
            this.agent.findBetterHouse();
        }

        public void visitHouse() {
            this.agent.visitHouse();
        }

        public void signContract() {
            this.agent.signContract();
        }

        public void findBetterHouseByFTX() {
            this.agent.findBetterHouse();
        }

        public void visitHouseByFTX() {
            this.agent.visitHouse();
        }

        public void signContractByFTX() {
            this.agent.signContract();
        }
    }


    public class FTXEstateAgent {
        public void findBetterHouse() {
            System.out.println("客官您好, 我们已经帮您找到了优质房源");
        }

        public void visitHouse() {
            System.out.println("客官, 我们现在带您参观一下房子");
        }

        public void signContract() {
            System.out.println("客官, 您满意我们现在就签购房合同");
        }
    }
```

同时Demo房产中介提供商也要做出相应的改动, 不能再调用以前的方法了. 而且改动起来牵动了整个上下层的改动; 经过几次麻烦之后, 中介提供商有点不耐烦了, 这不行啊, 每次给你找的中介不满意 还要费好大力气说动你接受另外一家房产中介, 太麻烦了. 这样吧, 既然你的目的是找一个优质房源, 我也不想这么折腾, 你干脆在心里接受所有的房产中介, 我可以给你找国内排行前10个房产中介为你服务, 你只需要根据中介的要求去看房 签合同就可以了, 我就不在说服你方面瞎折腾了. 我一想这样也可以, 避免了很大折腾, 我也少出点咨询费; 那么我现在要调整心理了。

```java
    public class Client {
        private Agent  agent;

        public void setAgent(Agent agent) {
            this.agent = agent;
        }

        public void findBetterHouse() {
            this.agent.findBetterHouse();
        }

        public void visitHouse() {
            this.agent.visitHouse();
        }

        public void signContract() {
            this.agent.signContract();
        }
    }
    
    public interface Agent {
        public void findBetterHouse();

        public void visitHouse();

        public void signContract();
    }
    
    public class LianJiaEstateAgent implements Agent {
        public void findBetterHouse() {
            System.out.println("客官您好, 我们已经帮您找到了优质房源");
        }

        public void visitHouse() {
            System.out.println("客官, 我们现在带您参观一下房子");
        }

        public void signContract() {
            System.out.println("客官, 您满意我们现在就签购房合同");
        }
    }

    public FTXEstateAgent implements Agent {
        public void findBetterHouse() {
            System.out.println("客官您好, 我们已经帮您找到了优质房源");
        }

        public void visitHouse() {
            System.out.println("客官, 我们现在带您参观一下房子");
        }

        public void signContract() {
            System.out.println("客官, 您满意我们现在就签购房合同");
        }
    }
```

通过接受所有房产中介, 那么只要能提供我要的服务的房产中介我都可以尝试使用; 如果后面发现了其他的中介, 我现在基本上不用做任何事情, 只需要重新找一个房产中介就可以使用我想要的服务了.这样底层实现也不需要变动, 只需要扩展Agent接口类就可以了. 同时高层方面只需要替换new 出来的房产中介的不同实例就可以了.把变更风险降到最低;

### 总结

依赖倒置的本质就是通过抽象使各个类、模块的实现彼此相互独立, 实现模块间的松耦合. 在开发过程中, 我们只要遵循几个规则便可以做到依赖倒置:

    1、每个类尽量都有接口或者抽象类
    2、在创建类的时候, 使用接口或者抽象类进行声明; 创建的时候可以使用具体的对象实例
    3、同时结合里式替换原则, 当替换之后的行为不改变; 由于接口具有相同的行为, 所以使用依赖倒置自然也就符合了里式替换原则;

