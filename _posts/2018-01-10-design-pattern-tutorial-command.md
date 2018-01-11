---
layout: post
title: 设计模式系列之命令模式
date: 2018-01-11 20:43:00
tags: [设计准则, 命令模式]
---

### 命令模式

用定义来说的话就是:命令模式把一个请求或者操作封装到一个对象(Command对象)中。命令模式允许系统使用不同的请求把客户端参数化，对请求排队或者记录请求日志，可以提供命令的撤销和恢复功能。

用比较技术的话来说的话(我的理解)就是 将命令的发起者与命令的执行者解耦, 依托于中间的协调者完成命令的发布与执行.

命令模式的通用类图如下:

![命令模式通用类图](/assets/images/2018-01-11-design-pattern-command.png)

该类图中主要有5个角色:

1、Invoker类: 负责指挥Command的执行

2、Command类: 负责定义Command的行为, 即execute()方法

3、ConcreteCommand类: 负责实现Command中的方法, 同时也是定义中提到的请求或者操作的封装.

4、Receiver: 命令的真正执行者的行为定义. 主要定义action方法.如果很多命令都是同一个对象执行, 那么可以定义多个行为

5、ConcreteReceiver: 实现action方法, 根据具体的命令实现不同的行为

6、客户端类: 主要是演示命令模式应该如何使用

### 应用场景

在实际项目中, 目前用到了命令模式的场景在我印象中只有一个地方. 是做的一个web界面的代码编辑器工作. 如果大家用过c9, 那么应该可以明白我说的是个什么东西. web版代码编辑器与本地的VS Code 或者任何编辑器或者IDE都有相同的一个功能: 文件的创建、文件夹的创建、文件内容的修改等等。下面我们根据该场景来开发

我们还是先看一些过程流或者单一类应该怎么写?

```java

public class CommandExecutor {

    public static void executeCommand(String command) {
        if ("创建文件".equals(command)) {
            System.out.println("创建文件成功");
        }else if ("创建文件夹".equals(command)) {
            System.out.println("创建文件夹成功");
        }else if("修改文件内容".equals(command)) {
            System.out.println("修改文件内容成功");
        }
    }
}

class Client {
    public static void main(String[] args) {
        CommandExecutor.executeCommand("创建文件");
        CommandExecutor.executeCommand("创建文件夹");
        CommandExecutor.executeCommand("修改文件内容");
    }
}
```

我们在上面程序中定义了一个命令执行者以及客户端调用示例. 我们发现客户端代码挺干净的, 唯一存在的问题就是用户需要知道命令的描述是什么.不太友好(我们将executeCommand的参数改为对象类型肯定更加友好).下面我们改一下代码:

```java
public class CommandExecutor {

    public static void executeCommand(CommandEnum command) {
        switch (command) {
            case CREATE_DIR:
                System.out.println("创建文件夹成功");
                break;
            case CREATE_FILE:
                System.out.println("创建文件成功");
                break;
            case UPDATE_FILE_CONTENT:
                System.out.println("修改文件内容成功");
                break;
            default:
                break;
        }
    }
}

enum CommandEnum {

    CREATE_FILE("创建文件"), CREATE_DIR("创建文件夹"), UPDATE_FILE_CONTENT("修改文件内容");

    private String command;

    private CommandEnum(String command) {
        this.command = command;
    }

    public String getCommand() {
        return command;
    }

    public void setCommand(String command) {
        this.command = command;
    }
}

class Client {
    public static void main(String[] args) {
        CommandExecutor.executeCommand(CommandEnum.CREATE_FILE);
        CommandExecutor.executeCommand(CommandEnum.CREATE_DIR);
        CommandExecutor.executeCommand(CommandEnum.UPDATE_FILE_CONTENT);
    }
}
``` 

上面代码我们增加了一个命令枚举型, 然后传入到CommandExecutor的接口中, 相当于为用户提供了一个很好的文档, 当用户不知道要传什么参数过去的时候, 可以直接打开CommandEnum类就可以看到所有的类型了. 这样设计确实比最开始的设计要好很多. 但是不可避免的在扩展方面存在限制. 我们要增加一个命令的时候需要修改Switch的代码, 因为我们这边只是简单的打印如果对于复杂逻辑维护性真的不可谓不大. 如果将Switch每一个分支下的逻辑处理放在另外一个类中分别由不同的方法实现也勉强可以接受. 但是我们有更好的方式去实现, 为什么不尝试下呢?

下面我们用命令模式来设计该场景的代码。首先因为我们知道命令分为3种, 分别为```创建文件``` ```创建文件夹``` ```更新文件内容```.

那么我们先开发命令接口:

```java
public interface Command {
    void execute();
}
```

然后创建一个Receiver接口, 因为我们的Command实现中需要该接口:

```java
public interface CommandReceiver {
    void action();
}
```

然后我们创建三种Command类:

创建文件命令类:

```java
public class CreateFileCommand implements Command {

    private CommandReceiver commandReceiver;

    public CreateFileCommand(CommandReceiver commandReceiver) {
        this.commandReceiver = commandReceiver;
    }

    public void execute() {
        this.commandReceiver.action();
    }
}

```

创建文件夹命令类:

```java
public class CreateDirCommand implements Command {

    private CommandReceiver commandReceiver;

    public CreateDirCommand(CommandReceiver commandReceiver) {
        this.commandReceiver = commandReceiver;
    }

    public void execute() {
        this.commandReceiver.action();
    }
}
```

更新文件命令类:

```java
public class UpdateFileCommand implements Command {

    private CommandReceiver commandReceiver;

    public UpdateFileCommand(CommandReceiver commandReceiver) {
        this.commandReceiver = commandReceiver;
    }

    public void execute() {
        this.commandReceiver.action();
    }
}
```

下面我们需要开发具体执行命令的Receiver类:

```java
public class CreateDirReceiver implements CommandReceiver {
    public void action() {
        System.out.println("创建文件夹成功");
    }
}

public class CreateFileReceiver implements CommandReceiver {
    public void action() {
        System.out.println("创建文件成功");
    }
}

public class UpdateFileReceiver implements CommandReceiver {
    public void action() {
        System.out.println("更新文件内容成功");
    }
}
```

下面我们开发Invoker对象:

```java
public class Involker {
    public void invoke(Command command) {
        command.execute();
    }
}
```

我们看一下客户端如何调用呢?

```java
public class Client {
    public static void main(String[] args) {
        Involker involker = new Involker();
        Command createFileCommand = new CreateFileCommand(new CreateFileReceiver());
        Command createDirCommand = new CreateDirCommand(new CreateDirReceiver());
        Command updateFileCommand = new UpdateFileCommand(new UpdateFileReceiver());

        involker.invoke(createDirCommand);
        involker.invoke(createFileCommand);
        involker.invoke(updateFileCommand);
    }

}
```

我们能够看到, 在新增一个命令的时候, 我们只需要分别实现相关接口, 然后就可以调用了. 遵循了开闭原则. 但是我们也能够看到这样的代码, 对于客户端来说, 客户端与下层的类之间耦合性非常强. 但是其实Invoker、Command和Receiver这三者才应该是内聚在一块的, 而Client应该只需要跟Invoker通信就可以了. 或者在增加一个辅助类, 来确定到底该用哪种命令与接收者.

这个时候我们我们可能增加一个BeanContainer就能够解决问题. BeanContainer与前面开闭原则那篇文章一样;

```java
import java.util.HashMap;
import java.util.Map;

public class BeanContainer {
    private final static Map<String, Command> beanContainerMap = new HashMap<String, Command>();

    static {
        beanContainerMap.put(CommandEnum.CREATE_FILE.getCommand(), new CreateFileCommand(new CreateFileReceiver()));
        beanContainerMap.put(CommandEnum.CREATE_DIR.getCommand(), new CreateDirCommand(new CreateDirReceiver()));
        beanContainerMap.put(CommandEnum.UPDATE_FILE_CONTENT.getCommand(), new UpdateFileCommand(new UpdateFileReceiver()));
    }

    public static Command getCommandByType(CommandEnum commandType) {
        return beanContainerMap.get(commandType.getCommand());
    }
}
```

然后我们客户端在调用的时候就只与Container通信就可以了.

```java
public class Client {
    public static void main(String[] args) {
        Involker involker = new Involker();
        Command createFileCommand = BeanContainer.getCommandByType(CommandEnum.CREATE_FILE);
        Command createDirCommand = BeanContainer.getCommandByType(CommandEnum.CREATE_DIR);
        Command updateFileCommand = BeanContainer.getCommandByType(CommandEnum.UPDATE_FILE_CONTENT);
        
        involker.invoke(createFileCommand);
        involker.invoke(createDirCommand);
        involker.invoke(updateFileCommand);
    }
}
```

我们通过静态工厂模式, 直接将需要内聚的2个类内聚在BeanContainer类里面, 而Client 与 Command的具体实现不具有任何的耦合. 

如果我们再分析一下会发现, 整个类结构还可以更加内聚. 整个命令的获取可以放入到invoke方法中.

```java
public class Invoker {
    public void invoke(CommandEnum commandEnum) {
        BeanContainer.getCommandByType(commandEnum).execute();
    }
}
```

那么我们的客户端就只需要使用如下方式调用命令了.

```java
public class Client {
    public static void main(String[] args) {
        Invoker involker = new Invoker();
        involker.invoke(CommandEnum.CREATE_FILE);
        involker.invoke(CommandEnum.CREATE_DIR);
        involker.invoke(CommandEnum.UPDATE_FILE_CONTENT);
    }
}
```
这样我们通过命令模式的设计 + 静态工厂改进, 直接将客户端调用与文章中最开始客户端调用部分做到了一样的效果. 但是可扩展性要比文章最开始要好的多.

### 实践心得

在实际开发过程中, 我们可以通过以上模式代码来完成 逻辑意义上具有命令概念在里面的一些场景的设计工作. 同时也可以适当的根据模板方法来减少命令类的爆炸情况. 举例: 在上面的代码中由于Command的子类具有相同的行为. 所以就可以直接使用一个类来完成抽象工作. 如果命令中存在不同的行为那么就要在模板类中增加abstract 的 execute方法来由子类实现了. 

当然在该文章中, 我们用到的Receiver具有多个, 所以Command的实现类只有一个就可以了, 因为Command实现类的行为完全一致.
如果在设计的时候Receiver只有一个, 那么Command的实现类就需要有多个, 因为每一个Command的execute方法都需要调用同一个Receiver的不同的方法才能保证命令执行的是不同的行为。
