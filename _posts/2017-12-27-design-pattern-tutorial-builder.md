---
layout: post
title: 设计模式系列之建造者模式
date: 2017-12-27 20:49:00
tags: [设计准则, 建造者模式]
---

### 建造者模式
 
 建造者模式将一个复杂对象的构建与表示分离，使得同样的构建过程可以创建不同的表示。建造者模式基本上就是将一个复杂对象的各个组件进行拼接, 使用相同的拼接流程但是却能够获得不同的产品; 当流水线上面的操作相同, 但是出来的产品不同时, 那么必然是产品中的零件不同了.

 建造者通用类图如下:

![建造者模式类图](/assets/images/2017-12-27-design-pattern-builder.png)

有人看到这个类图发现, 咦，乖乖, 怎么跟其他的博客中画的建造者模式的类图不太一样呢? 确实不一样, 因为其他的类图太抽象了. 理解起来如果没有代码辅助的话, 对于初学者很难理解. 所以我就自己画了一个类图;

在建造者模式中, 有如下6个角色:

Product接口: 约束产品中的有多少个零件; 通过定义接口中的setPartA等方法实现
ConcreteProduct类: 对于产品Product接口的具体实现;
Builder接口或者抽象类: 规范产品的组件, 一般有子类实现
ConcreteBuilder类: 负责具体的产品对象的构建
Director类: 负责安排产品的构建过程

上面整个对于建造者的描述虽然加入了通俗的语言描述, 但是还是很抽象的. 到底建造者模式怎么用? 在哪些场景下可以用?正是本文所要描述的.

### 应用场景

还是富士康工厂啊, 我好像对富士康工厂具有额外的感情, 每次举例都会想到他, 但其实在创建类型的设计模式来说, 工厂无疑是最贴切与恰当的. 前面我们讲了使用抽象工厂模式来创建苹果产品与小米产品, 现在我们要深入了解小米工厂里面具体要干什么事情了.

富士康工厂小米车间有一条生产线, 生产```红米手机```; 比如需要装配屏幕、电池、CPU、主板、螺丝钉; 我们就以这个过程来看看如何建模:

首先, 我们需要先创建产品类的接口 ```IPhone```:

    interface IPhone {
        void call();
        void surfOnline();
    }

然后, 我们创建产品的具体实现:

    class XiaomiPhone implements IPhone {
        private String cpu;
        private String screen;
        private String battery;
        private String mainBoard;
        private String screw;

        public void call() {
            System.out.println("主人, 我正处于通话中");
        }

        public void surfOnline() {
            System.out.println("主人, 我正在帮您上网");
        }

        public String getCpu() {
            return cpu;
        }

        public void setCpu(String cpu) {
            this.cpu = cpu;
        }

        public String getScreen() {
            return screen;
        }

        public void setScreen(String screen) {
            this.screen = screen;
        }

        public String getBattery() {
            return battery;
        }

        public void setBattery(String battery) {
            this.battery = battery;
        }

        public String getMainBoard() {
            return mainBoard;
        }

        public void setMainBoard(String mainBoard) {
            this.mainBoard = mainBoard;
        }

        public String getScrew() {
            return screw;
        }

        public void setScrew(String screw) {
            this.screw = screw;
        }
    }

这时候我们的产品类就已经创建完成了, 那么我们现在就要看工人们如何来一步步的完成这个手机的制作了;

我们创建Builder的接口类:

    interface IPhoneBuilder {
        void withCpu(String cpu);
        void withScreen(String screen);
        void withBattery(String battery);
        void withMainBoard(String mainBoard);
        void withScrew(String screw);

        IPhone build();
    }

下面, 我们创建```XiaomiPhoneBuilder```类:

    class XiaomiPhoneBuilder implements IPhoneBuilder {

        XiaomiPhone xiaomiPhone = new XiaomiPhone();

        public void withCpu(String cpu) {
            xiaomiPhone.setCpu(cpu);
        }

        public void withScreen(String screen) {
            xiaomiPhone.setScreen(screen);
        }

        public void withBattery(String battery) {
            xiaomiPhone.setBattery(battery);
        }

        public void withMainBoard(String mainBoard) {
            xiaomiPhone.setBattery(mainBoard);
        }

        public void withScrew(String screw) {
            xiaomiPhone.setScrew(screw);
        }

        public IPhone build() {
            return this.xiaomiPhone;
        }
    }

我们基本上完整了工人 与 产品的角色了, 还差一个指挥者的角色.

    class PhoneDirector {
        private IPhoneBuilder builder = new XiaomiPhoneBuilder();

        public IPhone createHongMiPhone() {
            builder.withMainBoard("主板");
            builder.withCpu("英特尔");
            builder.withBattery("飞毛腿");
            builder.withScreen("三星");
            builder.withScrew("小螺丝");

            return builder.build();
        }
    }

我们看一下客户端如何调用的:

    public class Client {
        public static void main(String[] args) {
            PhoneDirector phoneDirector = new PhoneDirector();
            IPhone phone = phoneDirector.createHongMiPhone();
            phone.call();
            phone.surfOnline();
        }
    }

这样我们就可以完成```红米```的创建过程了. 我们与现实流水线生产手机的对比:

    1、XiaomiPhone类相当于已经组装好的红米手机成品
    2、XiaomiPhoneBuilder相当于在流水线边上坐着的工人们.
    3、Director相当于工人们不知道怎么组装手机, 需要工段长来指挥完成.
    4、换句话说, Director也可以相当于流水线的位置, 处于流水线的最开始的位置的人负责安装主板, 处于最后一个位置的人负责拧紧螺丝, 最后装箱即可; 工业化生产基本上就根据流水线的位置, 工人们进行不同的操作;
    5、而我们的指挥者也扮演着与流水线相同的角色;

建造者模式的扩展性:

假设我们现在需要构建一个```小米4```的手机, 我们需要做的工作就是新建一个```小米4```的类, 并且实现```小米4建造者```， 同时实现一个```小米4构建过程的指挥者```, 可能```小米4```是需要先安装屏幕, 再安装电池呢, 那么我们的这种建造者设计模式也完全能够满足相应的需求;

### 实践心得

在实际的工作过程中, 我并没有碰到很符合该设计模式的使用场景, 因为PhoneDirector类中的方法属于硬编码进去的, 这基本上与使用抽象工厂来完成一个对象的创建, 并且对象创建的过程也可以在抽象工厂中体现. 所以, 我在工程中并没有使用该场景下的建造者模式. 但是对于一个参数很多情况下的对象, 使用建造者模式会给上层调用带来很大的便利性, 如果需要往该对象中增加一个属性, 我们也不需要重新写一个叠加的构造方法就能够满足需求, 客户端也只需要在链式调用的后面增加一个属性的设置即可. 比如:

    class User {
        private String name;
        private String address;
        private String email;
        private String role;

        public String getName() {
            return name;
        }

        public String getAddress() {
            return address;
        }

        public String getEmail() {
            return email;
        }

        public String getRole() {
            return role;
        }

        static class UserBuilder {
            private String name;
            private String address;
            private String email;
            private String role;

            public UserBuilder withName(String name) {
                this.name = name;
                return this;
            }

            public UserBuilder withAddress(String address) {
                this.address = address;
                return this;
            }

            public UserBuilder withEmail(String email) {
                this.email = email;
                return this;
            }

            public UserBuilder withRole(String role) {
                this.role = role;
                return this;
            }
            
            public User build() {
                User user = new User();
                user.name = this.name;
                user.address = this.address;
                user.email = this.email;
                user.role = this.role;
            }
        } 
    }

那么我们客户端在调用的时候, 就会是这样的:

    public class Client {
        public static void main(String[] args) {
            User user = new User.UserBuilder().withAddress("深圳")
                    .withEmail("1243@qq.com").withName("terry")
                    .withRole("管理员").build();
            System.out.println(user.getAddress());
        }
    }

那么, 如果我们需要增加一个属性值 age, 我们也只需要在User中增加一个age属性 与 get方法, 同时修改Builder类; 而上层调用仅仅只需要在build()调用前, 增加一个withAge()的调用即可. 虽然这种方式对于扩展并不是开放的, 但是尽可能的减少了由于增加字段而使得上层代码改动很大的情况. 同时该模型的好处就是隐藏了生成对象的set功能, 使用建造者产生了不可变对象. 对于系统功能的稳定性具有很大的帮助.





