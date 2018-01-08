---
layout: post
title: 设计模式系列之迪米特法则
date: 2017-12-21 20:49:00
tags: [设计准则, 迪米特法则]
---

### 迪米特法则

迪米特法则 应为 Law Of Demeter, 简写LOD; 也称最少知识原则. 迪米特法则描述了两层方面的意思, 一个对象应该对其他对象保持最少了解 第二: 被依赖的对象应该暴露最少的公共接口.

迪米特法则还有一种解释：Only talk to your immediate friends，即只与直接朋友通信.首先来解释编程中的朋友:两个对象之间的耦合关系称之为朋友,通常有依赖,关联,聚合和组成等.而直接朋友则通常表现为关联,聚合和组成关系,即两个对象之间联系更为紧密,通常以成员变量,方法的参数和返回值的形式出现. 如果一个对象只在方法体中出现, 那么很有可能说明这个对象不是 本类所必须依赖的, 就要看一下能够把它剔除出自己的朋友圈了.

### 场景示例

假设我们要去餐馆吃饭, 点一大份牛肉面, 少放盐、多放肉、多放面, 做得快一点赶时间. 我们就上面这么一点需求. 如果用程序来实现我们应该怎么实现呢?先来一个不符合迪米特法则的实现;

实现一个点菜员的角色OrderClerk:

```java
    class OrderClerk {
        public void order(String food) {
            System.out.println(food);
        }
    }
```

实现一个厨师的角色 Kitchener:

```java
    class Kitchener {
        public void cook(String food) {
            System.out.println(food);
        }
    }
```

实现客户端点餐程序:

```java
    public class Client {
        public static void main(String args[]) {
            String foodDescription = "来一份大份牛肉面";
            OrderClerk orderClerk = new OrderClerk();
            orderClerk.order(foodDescription);

            Kitchener kitchener = new Kitchener();
            kitchener.cook("少放盐不要葱花, 快一点啊");
        }
    }
```

我们在程序中 先跟服务员说来一份大份牛肉面, 然后又跟后厨说少放盐、不要放葱花; 这样客户端就与点餐员 和 后厨产生了耦合;这样并不符合迪米特法则, 实际上我们根本不需要跟后厨打交道, 我们只与我们需要打交道的对象打交道; 那么我们其实在饭店只需要跟点菜员打交道就可以了.下面我们修改程序, 来实现只与点餐员打交道:

首先, 重构点餐员的角色:

```java
    class OrderClerk {
        Kitchener kitchener;

        OrderClerk() {
            kitchener = new Kitchener();
        }

        public void order(String food) {
            kitchener.cook(food);
        }
    }
```

厨师代码不需要动, 我们看客户端程序应该怎么调用:

```java
    public class Client {
        public static void main(String args[]) {
            String foodDescription = "来一份大份牛肉面, 少放盐不要葱花, 快一点啊";
            OrderClerk orderClerk = new OrderClerk();
            orderClerk.order(foodDescription);
        }
    }
```

通过重构我们让客户端只依赖了最直接的朋友```点菜员```. 这样就符合迪米特法则的第一个要求了, 客户端只与最直接的朋友打交道;

第二个要求也很好理解, 就不写代码举例了. 如果这个```点菜员```也会唱歌, 实现的时候 同样实现了唱歌的Public方法, 那么客户端在点菜的时候自然也能看到```唱歌```这个能力了, 但是我就想安安静静吃个饭, 要什么自行车。。所以实现代码的时候同样要符合迪米特法则的第二个要求: 被依赖的类尽量暴露最少的公共方法;

比如在餐馆服务员只需要点餐, 但是服务员在KTV也会唱歌; 这样也好办..基于接口编程完全能够解决这个问题; 在餐馆, 我们依赖IOrder接口实例化出来OrderClerk对象来点餐, 在KTV场景下, 我们依赖ISing接口, 实例化出来OrderClerk对象来唱歌. 因为接口隔离所以 我们在餐厅场合 也看不到 服务员会唱歌的方法了. 

所以在开发程序的时候, 基于 被依赖者尽量少公开不必要的方法, 对于调用者尽量不依赖自己的不需要依赖的类 基本上就可以让程序符合迪米特法则. 迪米特法则无处不在, 所以对于该规则的理解还需要在写程序的时候不断思考场景才能理解的更加透彻与运用的更加熟练; 
