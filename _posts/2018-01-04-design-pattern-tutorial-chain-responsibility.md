---
layout: post
title: 设计模式系列之责任链模式
date: 2018-01-04 22:43:00
tags: [设计准则, 责任链模式]
---

### 责任链模式

责任链模式定义: 使多个对象都有机会处理请求, 从而避免了请求的发送者和接受者之间的耦合关系. 将这些对象连成一条链, 并沿着这条链传递该请求, 直到有对象处理它为止.

责任链模式的类图:

![责任链模式类图](/assets/images/2018-01-04-design-pattern-chain-responsibility.png)

该类图中主要包含2个角色:

1、抽象的处理类Handler: 主要用于定义抽象类的行为, 与成员变量nextHandler

2、具体的Handler类: 实现链上某一个节点的处理逻辑

### 应用场景

在公司里面基本上都会有请假的流程. 当然在我目前所在的公司是以电子流的形式体现的. 具体的规则是这样的

1、如果一次请假1天由PM审批
2、如果一次请假3天以内, 最最小部门的部长审批
3、如果一次请假5天以内, 以最小部门的上一级主管审批
4、如果一次请假5天以上, 需要由部门总裁审批

我们基于上面的请求开始编写代码, 我们先定义一个抽象类, 用于定义nextHandler 以及处理请求的抽象接口:

```java
    public abstract class LeaveRequestHandler {
        protected LeaveRequestHandler nextRequestHandler;

        public void setNextRequestHandler(LeaveRequestHandler nextRequestHandler) {
            this.nextRequestHandler = nextRequestHandler;
        }

        public abstract String handlerLeaveRequest(int leaveDays);
    }
```

然后我们分别根据需求定义PM处理类

```java
    public class PMLeaveRequestHandler extends LeaveRequestHandler {

        public String handlerLeaveRequest(int leaveDays) {
            if (leaveDays <= 1) {
                return "Project Manager approve your leave request";
            }

            if (this.nextRequestHandler != null) {
                return this.nextRequestHandler.handlerLeaveRequest(leaveDays);
            }

            return "Project Manager reject your leave request";
        }
    }
```

定义最小部门部长处理类:

```java
    public class DLLeaveRequestHandler extends LeaveRequestHandler {
        public String handlerLeaveRequest(int leaveDays) {
            if (leaveDays <= 3) {
                return "Department Leader approve your leave request";
            }

            if (this.nextRequestHandler != null) {
                return this.nextRequestHandler.handlerLeaveRequest(leaveDays);
            }

            return "Department Leader reject your leave request";
        }
    }
```

定义上一级部门部长处理类:

```java
    public class UpperDLLeaveRequestHandler extends LeaveRequestHandler {
        public String handlerLeaveRequest(int leaveDays) {
            if (leaveDays <= 5) {
                return "Upper Department Leader approve your leave request";
            }

            if (this.nextRequestHandler != null) {
                return this.nextRequestHandler.handlerLeaveRequest(leaveDays);
            }

            return "Upper Department Leader reject your leave request";
        }
    }
```

定义总裁处理类:

```java
    public class VPLeaveRequestHandler extends LeaveRequestHandler {
        public String handlerLeaveRequest(int leaveDays) {

            if (leaveDays > 5 && leaveDays < 15) {
                return "VP have approved your leave request";
            }

            if (this.nextRequestHandler != null) {
                return this.nextRequestHandler.handlerLeaveRequest(leaveDays);
            }

            return "VP reject your leave request";
        }
    }
```

这样对于责任链上的各个类已经完成了, 我们看一下客户端应该如何调用呢?

```java
    public class Client {
        public static void main(String[] args) {
            LeaveRequestHandler pmLeaveRequestHandler = new PMLeaveRequestHandler();
            LeaveRequestHandler dlLeaveRequestHandler = new DLLeaveRequestHandler();
            LeaveRequestHandler upperDLLeaveRequestHandler = new UpperDLLeaveRequestHandler();
            LeaveRequestHandler vpLeaveRequestHandler = new VPLeaveRequestHandler();

            pmLeaveRequestHandler.setNextRequestHandler(dlLeaveRequestHandler);
            dlLeaveRequestHandler.setNextRequestHandler(upperDLLeaveRequestHandler);
            upperDLLeaveRequestHandler.setNextRequestHandler(vpLeaveRequestHandler);

            String result = pmLeaveRequestHandler.handlerLeaveRequest(1);
            System.out.println(result);

            result = pmLeaveRequestHandler.handlerLeaveRequest(2);
            System.out.println(result);

            result = pmLeaveRequestHandler.handlerLeaveRequest(3);
            System.out.println(result);

            result = pmLeaveRequestHandler.handlerLeaveRequest(4);
            System.out.println(result);

            result = pmLeaveRequestHandler.handlerLeaveRequest(5);
            System.out.println(result);

            result = pmLeaveRequestHandler.handlerLeaveRequest(6);
            System.out.println(result);

            result = pmLeaveRequestHandler.handlerLeaveRequest(45);
            System.out.println(result);
        }
    }
```

我们通过打印可以看到, 对于不同场景下的请求都有不同的处理类来进行处理:

```java
    Project Manager approve your leave request
    Department Leader approve your leave request
    Department Leader approve your leave request
    Upper Department Leader approve your leave request
    Upper Department Leader approve your leave request
    VP have approved your leave request
    VP reject your leave request
```

这样我们就完成了整个责任链模式的使用.

### 实践心得

责任链模式比较适合使用在需要好多的类对请求进行处理, 处理类对于处理的场景又有比较明确的划分的情况下, 使用责任链模式是一个比较好的选择.



