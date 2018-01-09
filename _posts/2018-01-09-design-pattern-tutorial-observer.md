---
layout: post
title: 设计模式系列之观察者模式
date: 2018-01-09 21:30:00
tags: [设计准则, 观察者模式]
---

### 观察者模式

观察者模式又叫发布订阅模式; 该模式主要是解决当一个对象发生变化时, 能够及时通知与他有关系的对象都发生更新操作.

观察者模式在实际项目开发中经常会使用到的一种模式, 比如在系统开发过程中: XMPP中的presence报文作用、MQTT中的基于发布订阅的通信、iOS中的Notification的使用等等, 使用互联网产品的youtube的subscribe功能, QQ的好友上线提醒功能. 以及在生活中买东西, 如果货还没到, 那么你登记一下, 货到了通知你. 这个也是另外一种形式的观察者模式.

说了这么多场景, 我们来看一下观察者模式的通用类图:

![观察者模式通用类图](/assets/images/2018-01-09-design-pattern-observer.png)

### 应用场景

 举一个在移动领域比较常见的场景. 在微信聊天里面, 当我们收到某个好友的消息的时候, 微信程序需要帮我们更新消息列表界面的数据(消息界面), 同时还需要帮我们更新聊天室里面的数据刷新工作. 我们就模拟该场景来实现观察者模式。

 首先, 我们知道如果要接收消息, 那么必定有一个SockerListener, 这个类是用来监听socket套接字里面的内容, 通过二进制私有协议转换为程序中定义的Message实体. 而这个SocketListener就是我们类图中的ConcreteSubject类

我们基于面向接口编程的思想, 还是先定义一个Subject接口类

 ```java
 public interface SocketListener {
    //这里为简单起见, 我们使用字符串来表示Message实体
    void receiveMessage(String message);

    void registerObserver(Observer observer);

    void unRegisterObserver(Observer observer);
}

```

我们创建一个ConcreteSubject类

```java
import java.util.ArrayList;
import java.util.List;

public class DefaultSocketListener implements SocketListener {

    private List<Observer> observerList = new ArrayList<Observer>(10);

    public void registerObserver(Observer observer) {
        observerList.add(observer);
    }

    public void unRegisterObserver(Observer observer) {
        observerList.remove(observer);
    }

    public void receiveMessage(final String message) {
        observerList.forEach(observer -> observer.handle(message));
    }
}
```

同时, 我们添加一个Observer的接口并给与实现

```java
public interface Observer {
    void handle(String message);
}

class MessageListObserver implements Observer {
    public void handle(String message) {
        System.out.println("更新消息界面, 如果消息界面没有该会话, 则增加一条会话, 如果存在则更新该会话的最新聊天记录");
    }
}

class ChatRoomObserver implements Observer {
    public void handle(String message) {
        System.out.println("聊天界面增加一条消息, 并且滚动条移动到聊天的最下面, 显示最新的消息");
    }
}
```

那么我们客户端应该如何调用呢?

```java
public class Client {

    public static void main(String[] args) {
        SocketListener socketListener = new DefaultSocketListener();
        socketListener.registerObserver(new ChatRoomObserver());
        socketListener.registerObserver(new MessageListObserver());

        socketListener.receiveMessage("Hello, 小明你好");
    }
}
```

当SocketListener接收到一条消息的时候, 则会通知其他需要做出动作的类进行更新操作, 包括更新界面或者更新数据库还是其他什么操作.

当然, 观察者模式同样可以反过来, 在Observer中增加subscribe方法, 来将自身注册到Subject中. 比如Observer声明一个一个订阅方法并实现

```java
public interface Observer {
    void handle(String message);
    void subscribe(SocketListener socketListener);
}

class MessageListObserver implements Observer {
    public void handle(String message) {
        System.out.println("更新消息界面, 如果消息界面没有该会话, 则增加一条会话, 如果存在则更新该会话的最新聊天记录");
    }

    public void subscribe(SocketListener socketListener) {
        socketListener.registerObserver(this);
    }
}

class ChatRoomObserver implements Observer {
    public void handle(String message) {
        System.out.println("聊天界面增加一条消息, 并且滚动条移动到聊天的最下面, 显示最新的消息");
    }

    public void subscribe(SocketListener socketListener) {
        socketListener.registerObserver(this);
    }
}
```

那么客户端就可以这样来调用了也可以达到相同的效果:

```java
public class Client {

    public static void main(String[] args) {
        SocketListener socketListener = new DefaultSocketListener();
        Observer chatRoomObserver = new ChatRoomObserver();
        chatRoomObserver.subscribe(socketListener);
        Observer messageListObserver = new MessageListObserver();
        messageListObserver.subscribe(socketListener);

        socketListener.receiveMessage("Hello, 小明你好");
    }
}
```

### 实践心得

在观察者模式中, Subject与Observer都是通过接口来交互的. 增加了扩展能力.上面的模板确实也是真实场景中可能用到的代码. 而在分别调用每一个观察者的handle方法的时候最好使用异步来调用, 避免一个观察者消耗时间过长而导致下一个观察者无法执行的情况.基本上在一个状态改变会引起一系列的对象需要更新的场景中都可以尝试使用观察者模式来实现.

当然在Java中也提供了Observer 和 Observable方法来为Java中实现观察者模式提供抽象的实现. 这两个类也可以好好利用.

Observer 表示观察者, 而Observable表示可以被观察的对象,即Subject对象.


