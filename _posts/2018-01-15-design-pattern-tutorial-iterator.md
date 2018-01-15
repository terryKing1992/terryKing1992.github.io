---
layout: post
title: 设计模式系列之迭代器模式
date: 2018-01-15 20:15:00
tags: [设计准则, 迭代器模式]
---

### 迭代器模式

迭代器模式（Iterator），提供一种方法顺序访问一个聚合对象中的各种元素，而又不暴露该对象的内部表示。

迭代器模式的通用类图表示:

![迭代器通用类图](/assets/images/2018-01-15-design-pattern-iterator.png)

该类图中包含4个主要类:

1、Iterator类: 主要定义迭代器的主要行为

2、ConcreteIterator类: 定义迭代器的实现

3、Aggregate: 定义集合类, 包括增、删、改、查的功能。

4、ConcreteAggregate: 定义具体的集合实现类

### 应用场景

因为在JDK的集合类中, 基本上都会实现迭代器类. 所以迭代器具体实现可以查看JDK中集合的实现. 当然为了加深印象, 现在也实现一个超级简单的迭代器, 只是为了实践迭代器模式的实现而已.

首先, 定义一个集合类:

```java
public interface Aggregate {
    void insert(String element, int index);
    void delete(String element);
    String get(int index);
    int size();
}
```

定义迭代器接口定义:

```java
public interface Iterator {
    boolean hasNext();
    String next();
}
```

定义集合类的实现类:

```java
import java.util.Vector;

public class ConcreteAggregate implements Aggregate {

    Vector<String> elements = new Vector<String>(4);

    public void insert(String element, int index) {
        elements.insertElementAt(element, index);
    }

    public void delete(String element) {
        elements.remove(element);
    }

    public String get(int index) {
        return elements.get(index);
    }

    public int size() {
        return this.elements.size();
    }

    public ConcreteIterator iterator() {
        return new ConcreteIterator(this);
    }

    class ConcreteIterator implements Iterator {

        private Aggregate aggregate;

        private int cursor;

        ConcreteIterator(Aggregate aggregate) {
            this.cursor = 0;
            this.aggregate = aggregate;
        }

        public boolean hasNext() {
            if (this.cursor < this.aggregate.size()) {
                return true;
            }
            return false;
        }

        public String next() {
            return this.aggregate.get(cursor++);
        }
    }
}

```

那么我们看一下客户端怎么调用呢?

```java

public class Client {
    public static void main(String[] args) {
        Aggregate aggregate = new ConcreteAggregate();
        for (int index = 0; index < 10; index++) {
            aggregate.insert("Hello" + index, index);
        }

        Iterator iterator = aggregate.iterator();

        while(iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}

```

我们可以通过迭代器简单的遍历整个集合中的元素, 但是又不把集合的内部细节暴露给客户端.

### 实践心得

迭代器模式在JDK的使用中非常常见, 但是目前已经不会实现自己的迭代器 以及 自己的容器了, 一般使用JDK自带的容器进行组合就可以了.