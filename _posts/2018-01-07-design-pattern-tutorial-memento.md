---
layout: post
title: 设计模式系列之备忘录模式
date: 2018-01-07 21:43:00
tags: [设计准则, 备忘录模式]
---

### 备忘录模式

备忘录模式: 在不破坏封装性的前提下, 捕获一个对象的内部状态并且在该对象之外保存这个状态。这样以后就可以将该对象恢复到原先保存的状态. 备忘录模式就是一个对象的备份模式, 提供了一种程序数据的备份方法。

备忘录模式类图如下:

![备忘录模式类图](/assets/images/2018-01-07-design-pattern-memento.png)

上述类图中主要包括3个主要类:

1、Originator类: 记录当前时刻的内部状态, 负责定义哪些状态属于备份的范围, 并且负责创建备忘录数据 以及 从备忘录数据中恢复到原始状态

2、Memento类: 用于备份Originator的内部状态, 在需要的时候提供给recovery方法用于Originator的内部状态的恢复.

3、CareTaker类: 负责保存Memento备忘类的对象, 也可以负责保存多个备忘录对象.

### 应用场景

备忘录模式的应用场景比较广泛, 像PhotoShop中的历史记录功能、编辑器的回退功能、数据库中的事务回滚功能、系统上下时候的历史war包备份、 Git中节点的回滚与遴选功能等等.

那么, 我们针对于上述场景中的上个版本的war包备份功能来做一个简单的备忘录模式的模拟;

首先, 有上线经验的同学应该知道, 每次上线我们都需要有一个上线异常的恢复方案. 当然也就是备份以前的war包, 用于上线失败之后的功能回滚. 其中有以下几个步骤:

1、将上个版本的war包进行备份; 当然一般会使用 mv XXX.war XXX.war.bak 命令

2、将新版本的工程打成war包 并且上传到服务器上

3、将war包放入到tomcat的webapps下面

4、进行验证, 如果验证成功那么上线成功; 如果验证失败但是又过了上线时间窗口那么必须回滚

5、将上个版本的war包重新放入到tomcat的webapps下面

那么我们类比一下会发现, 
1、对于上个版本的war包的备份就相当于是从Originator中创建了一个备忘录类, 该备忘录完全复制了上个版本的二进制包. 

2、而mv命令就相当于 Originator中的createMemento方法. 

3、CareTaker相当于上线变更人员, 负责执行mv命令的人.并且将备份拷贝到自己知道的一个目录下面. 

4、而3步骤中的备份就相当于CareTaker中的Memento对象.

5、如果上线失败, 需要进行回滚, 那么重新将上个版本的war包放入到webapps下相当于 Originator方法的recovery方法. 用于tomcat中该服务的原始服务的恢复.

那么我们开始针对如上场景进行编程. 假设War包中包括以下信息: 服务的端口、服务的API列表、服务的上下文。

我们先创建一个Originator类, 该类中维护了War包的一些状态:

```java
import java.util.ArrayList;
import java.util.List;

public class ServiceWarOriginator {

    private String contextPath;
    private Integer serverPort;
    private List<String> servicesList = new ArrayList<String>();


    public ServiceWarMemento createMemento() {
        List<String> copyServicesList = new ArrayList<String>(this.servicesList.size());
        copyServicesList.addAll(this.servicesList);
        ServiceWarMemento serviceWarMemento = new ServiceWarMemento(this.contextPath, this.serverPort, copyServicesList);
        return serviceWarMemento;
    }

    public void recoveryServiceWar(ServiceWarMemento serviceWarMemento) {
        this.contextPath = serviceWarMemento.getContextPath();
        this.serverPort = serviceWarMemento.getServerPort();

        List<String> copyServicesList = new ArrayList<String>(this.servicesList.size());
        copyServicesList.addAll(serviceWarMemento.getServicesList());
        this.servicesList = copyServicesList;
    }

    public String getContextPath() {
        return contextPath;
    }

    public void setContextPath(String contextPath) {
        this.contextPath = contextPath;
    }

    public Integer getServerPort() {
        return serverPort;
    }

    public void setServerPort(Integer serverPort) {
        this.serverPort = serverPort;
    }

    public void addService(String serviceName) {
        this.servicesList.add(serviceName);
    }
}
```

当然我们也需要创建一个备忘录类, 用于存储原来数据的状态.

```java
import java.util.List;

public class ServiceWarMemento {
    private String contextPath;
    private Integer serverPort;

    private List<String> servicesList;

    public ServiceWarMemento(String contextPath, Integer serverPort, List<String> servicesList) {
        this.contextPath = contextPath;
        this.serverPort = serverPort;
        this.servicesList = servicesList;
    }

    public String getContextPath() {
        return contextPath;
    }

    public Integer getServerPort() {
        return serverPort;
    }

    public List<String> getServicesList() {
        return servicesList;
    }
}
```

最后, 我们需要实现一个责任人类, 负责完成备忘录的存储, 就是变更人员要知道保存的上个版本的War包放在了哪个目录下

```java
public class WarCareTaker {
    private ServiceWarMemento serviceWarMemento;

    public WarCareTaker(ServiceWarMemento serviceWarMemento) {
        this.serviceWarMemento = serviceWarMemento;
    }

    public ServiceWarMemento getServiceWarMemento() {
        return serviceWarMemento;
    }

    public void setServiceWarMemento(ServiceWarMemento serviceWarMemento) {
        this.serviceWarMemento = serviceWarMemento;
    }
}
```

我们看一下如上实现的代码能够完成版本上线之后的回滚操作呢?

```java
public class Client {
    public static void main(String []args) {
        ServiceWarOriginator serviceWarOriginator = new ServiceWarOriginator();
        serviceWarOriginator.setContextPath("/serverless/v1");
        serviceWarOriginator.setServerPort(9080);
        serviceWarOriginator.addService("登录");
        serviceWarOriginator.addService("注册");

        System.out.println("当前上下文根为" + serviceWarOriginator.getContextPath() + ", 端口为" + serviceWarOriginator.getServerPort());
        System.out.println("当前版本的功能为" + serviceWarOriginator.getServices());

        //现在我们要新上一个功能, 用户注销功能, 同时由于版本变动,
        // 上下文根需要变为/serverless/v2。在部署之前, 我们要先将上个版本备份一下
        WarCareTaker warCareTaker = new WarCareTaker(serviceWarOriginator.createMemento());
        serviceWarOriginator.setContextPath("/serverless/v2");
        serviceWarOriginator.addService("注销用户");

        System.out.println("当前上下文根为" + serviceWarOriginator.getContextPath() + ", 端口为" + serviceWarOriginator.getServerPort());
        System.out.println("当前版本的功能为" + serviceWarOriginator.getServices());

        //当我们发现如果服务不可用的情况下, 我们需要回滚到上个版本的功能.
        serviceWarOriginator.recoveryServiceWar(warCareTaker.getServiceWarMemento());
        System.out.println("当前上下文根为" + serviceWarOriginator.getContextPath() + ", 端口为" + serviceWarOriginator.getServerPort());
        System.out.println("当前版本的功能为" + serviceWarOriginator.getServices());
    }
}
```

这样我们就完成了一个备忘录模式的使用, 我们看日志输出就会发现, 系统回到了原始版本

```
当前上下文根为/serverless/v1, 端口为9080
当前版本的功能为[登录, 注册]
当前上下文根为/serverless/v2, 端口为9080
当前版本的功能为[登录, 注册, 注销用户]
当前上下文根为/serverless/v1, 端口为9080
当前版本的功能为[登录, 注册]
```

### 实践心得

备忘录模式有很多的变形方式, 本文中只是介绍了最为通用的备忘录模式, 每种方式都有自己的优缺点, 需要开发者根据经验来实施具体的备忘录变形模式。这个就需要多看源码来比较了哪种备忘录模式适合于哪种场景了.

在有些场景下即使需要备份有些数据的状态, 但是在创建备忘录与恢复备忘录的过程都是异步的过程或者根本不在同一个系统中的时候, 也不可以强套备忘录模式实现, 毕竟都已经生成了不同的备忘录对象与Originator对象了.

但是在单机程序中, 比如PS的操作历史记录、Git的节点回滚与遴选操作、代码编辑器的撤销操作都可以使用备忘录模式实现。