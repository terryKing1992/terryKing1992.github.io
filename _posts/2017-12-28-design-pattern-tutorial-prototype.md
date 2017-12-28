---
layout: post
title: 设计模式系列之原型设计模式
date: 2017-12-28 20:12:00
tags: [设计准则, 原型设计模式]
---

### 原型模式

原型模式: 用原型实例指定创建对象的种类, 并且通过拷贝这些原型创建新的对象, 创建的新的对象与他的原型具有相同的属性与行为;
而原型模式的核心就是一个clone方法, 通过调用该方法实现对象的拷贝. 在Java中, 对象如果要能够调用clone方法, 需要实现Cloneable接口, Cloneable接口用来标示该对象是可以拷贝的.

原型模式类图如下:

![原型模式类图](/assets/images/2017-12-28-design-pattern-prototype.png)

### 应用场景

    1、当初始化的对象基本上属性都是一样的, 只有极个别的属性不一致的情况下可以使用克隆模式;
    2、当一个对象创建完成之后, 该对象需要被传入到多个方法中, 并且多个方法都会设置其属性值的情况下, 为了避免副作用, 使用原型模式克隆出一个完全一样的对象是比较好的

假设我们现在有一个群发邮件的需求. 我们应该如何实现呢? 当一个用户群发了一封邮件, 邮件的发件人, 内容一模一样, 只有收件人不一样, 这样的话, 服务器在收到发件人的发邮件请求的时候如何处理呢?

首先, 我们先定义一个EmailMessage类：

    class EmailMessage {
        private String from;
        private String to;
        private String body;

        public EmailMessage(String from, String to, String body) {
            this.from = from;
            this.to = to;
            this.body = body;
        }

        public String getFrom() {
            return from;
        }

        public void setFrom(String from) {
            this.from = from;
        }

        public String getTo() {
            return to;
        }

        public void setTo(String to) {
            this.to = to;
        }

        public String getBody() {
            return body;
        }

        public void setBody(String body) {
            this.body = body;
        }
    }

当服务器收到群发有件请求的时候, 我们通过分析发件人的模型, 解析出来了发件人 与 收件人列表 以及 邮件内容信息, 那么后面我们怎么处理呢? 如果没有原型模式, 我们需要每次都创建一个EmailMessage对象, 并且调用底层的发送方法; 代码实现如下:

    public static void main(String[] args) {
        String from = "terry";
        List<String> toList = new ArrayList<String>(1000000);
        for (int index = 0; index < 1000; index++) {
            toList.add("to" + index);
        }

        sendToAll(from, toList, "你们好, 我是新来的小明");
    }

    public static void sendToAll(String from, List<String> toList, String content) {
        long startTime = System.currentTimeMillis();
        for (String to : toList) {
            EmailMessage emailMessage = new EmailMessage(from, to, content);
            sendMessage(emailMessage);
        }
        long endTime = System.currentTimeMillis();

        System.out.println("总耗时为" + (endTime - startTime));
    }

    public static boolean sendMessage(EmailMessage emailMessage) {
        System.out.println("发送由发件人" + emailMessage.getFrom() + "发送给" + emailMessage.getTo() + "的邮件");
        return true;
    }

同样, 由于我们知道邮件对象的一些属性都是不变的, 只有```to```属性可能会发生变化, 所以我们也可以改为原型模式来复制对象, 并修改to的值, 然后调用```sendMessage```方法来实现.

    public static void sendToAll(String from, List<String> toList, String content) {
        long startTime = System.currentTimeMillis();
        EmailMessage emailMessagePrototype = new EmailMessage(from, "", content);

        for (String to : toList) {
            EmailMessage emailMessage = emailMessagePrototype.clone();
            emailMessage.setTo(to);
            sendMessage(emailMessage);
        }
        long endTime = System.currentTimeMillis();

        System.out.println("总耗时为" + (endTime - startTime));
    }

因为我们的数据量很小, 而且对象的创建也不复杂, 所以在性能上面没有太突出的表现. 不过当我们创建结构复杂的对象时, 确实可以考虑使用原型模式, 毕竟是本地方法调用的直接内存拷贝; 在大数据对象上应该会有不错的表现.

### 深复制与浅复制

在对象复制过程中, 如果是基本数据类型的属性或者String类型的属性, 都可以通过调用父类的clone方法完成对象的复制, 但是如果某一个对象中存在引用类型, 那么使用父类的clone方法会造成浅复制的情况, 以至于两个对象之间会出现引用一个实例的情况, 引发一下很难发现的Bug.

我们画一张图来表示浅复制的形成过程:

![浅复制](/assets/images/2017-12-28-design-pattern-shallow-copy.png)
 
由图中我们可以看出, 对象A里面有一个对象B的引用, 如果对象A直接调用父类的Clone方法, 那么产生的克隆对象A里面的引用也会指向对象, 从而造成对于```对象A```与```克隆对象A```对```对象B```的修改都会反映到对方上面。

下面我们看一个例子, 来看浅复制是如何进行的:

    class EmailMessage implements Cloneable {
        private String from;
        private String to;
        private String body;
        private Date sendDate;

        public EmailMessage(String from, String to, String body, Date sendDate) {
            this.from = from;
            this.to = to;
            this.body = body;
            this.sendDate = sendDate;
        }

        public String getFrom() {
            return from;
        }

        public void setFrom(String from) {
            this.from = from;
        }

        public String getTo() {
            return to;
        }

        public void setTo(String to) {
            this.to = to;
        }

        public String getBody() {
            return body;
        }

        public void setBody(String body) {
            this.body = body;
        }

        public Date getSendDate() {
            return sendDate;
        }

        public void setSendDate(Date sendDate) {
            this.sendDate = sendDate;
        }

        public EmailMessage clone() throws CloneNotSupportedException {
            return (EmailMessage) super.clone();
        }
    }

我们还拿上面的例子来举证, 并且在该类中增加一个引用数据类型的属性Date, 如果按照上面的写法, 因为是调用的父类的Clone方法, 所以只能完成浅复制的动作; 那具体的表现是怎样的呢? 我们通过一个客户端调用来看一下:

    public static void main(String[] args) throws CloneNotSupportedException {
        String from = "terry";
        String to = "yrret";
        String content = "Hello";
        Date date = new Date();

        EmailMessage emailMessage = new EmailMessage(from, to, content, date);
        EmailMessage copyEmailMessage = emailMessage.clone();

        System.out.println("查看经过Clone产生的对象中的引用类型的对象是否指向同一个真实对象:" + (emailMessage.getSendDate() == copyEmailMessage.getSendDate()));
        //我们修改emailMessage中sendDate对象的值, 看是否会影响到copyEmailMessage
        System.out.println("修改emailMessage前 的时间为" + emailMessage.getSendDate());
        System.out.println("copyEmailMessage的时间为" + copyEmailMessage.getSendDate());

        emailMessage.getSendDate().setTime(9384327432L);
        System.out.println("emailMessage 修改后的时间为" + emailMessage.getSendDate());
        System.out.println("copyEmailMessage在emailMessage修改后的时间为" + copyEmailMessage.getSendDate());
    }

我们运行程序可以发现, 对于原型中属性的修改确实影响到了复制体上, 运行程序可以看到打印结果为:

    查看经过Clone产生的对象中的引用类型的对象是否指向同一个真实对象:true
    修改emailMessage前 的时间为Thu Dec 28 21:18:47 CST 2017
    copyEmailMessage的时间为Thu Dec 28 21:18:47 CST 2017
    emailMessage 修改后的时间为Sun Apr 19 22:45:27 CST 1970
    copyEmailMessage在emailMessage修改后的时间为Sun Apr 19 22:45:27 CST 1970

举个栗子说明这种情况, 科学家根据人类细胞克隆出来一个一模一样的人, 但是科学家克隆的时候对于心脏等器官使用了浅复制, 那么会出现什么情况呢? 复制体被打的心脏破裂之后, 克隆人本体也完蛋了. 那么本体应该要骂娘了, "**, TMD, 本来想我生病了可以从他身上摘下来器官来用呢, 他妈的克隆人打架把我给搞死了". 显然这种并不是我们想要的结果;

那么应该如何才能实现深复制呢?

我们只需要将上面的clone方法简单修改一下即可:

     public EmailMessage clone() throws CloneNotSupportedException {
        EmailMessage copy = (EmailMessage) super.clone();
        copy.sendDate = (Date)this.sendDate.clone();
        return copy;
    }

对于引用类型的对象, 我们在克隆方法中再次调用一次克隆, 而此时我们再次运行客户端代码, 查看输出:

    查看经过Clone产生的对象中的引用类型的对象是否指向同一个真实对象:false
    修改emailMessage前 的时间为Thu Dec 28 21:27:02 CST 2017
    copyEmailMessage的时间为Thu Dec 28 21:27:02 CST 2017
    emailMessage 修改后的时间为Sun Apr 19 22:45:27 CST 1970
    copyEmailMessage在emailMessage修改后的时间为Thu Dec 28 21:27:02 CST 2017

这样我们就完成了深复制的功能;

### 实践心得

在实际的项目中, 对于原型模式的使用并不常见. 至少目前我还没怎么使用过原型模式. 对于具体的项目中的应用等什么时候阅读其他人源码的时候再补充吧~

