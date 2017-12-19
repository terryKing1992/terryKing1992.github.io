---
layout: post
title: 设计模式系列之里式替换原则
date: 2017-12-19 21:40:00
tags: [设计准则, 里式替换原则]
---

### 里氏替换原则

里式替换原则是Liskov于1987年提出了一个关于继承的原则; 它的定义如下: 如果对每一个类型为T1的对象o1，都有类型为T2的对象o2，使得以T1定义的所有程序P在所有的对象o1都换成o2时，程序P的行为没有变化，那么类型T2是类型T1的子类型。 对于公式的推导大部分人看懂了也不太容易记下来, 上面的定义感觉更像是公里或者定理. 而一个通熟易懂的解释更加让人容易接受跟理解; 通俗的定义: 在程序中, 任何使用基类的地方都可以透明的使用子类进行替换, 替换之后程序并不会出现什么错误或者异常情况.

### 里式替换原则在程序中的表现

##### 1、子类只实现基类中的抽象方法, 而不要去重写基类的非抽象方法

举个🌰: 如果我们在一些想不到的情况下, 需要继承ArrayList类, 而且重写了subList方法

    public class MyList<T> extends ArrayList<T> {
        @Override
        public List<T> subList(int fromIndex, int toIndex) {
            ArrayList<T> subList = new ArrayList<T>(toIndex - fromIndex);
            for (int index = fromIndex; index < toIndex; index++) {
                subList.add(this.get(index));
            }

            return subList;
        }
    }

那么会出现什么情况呢? 我们客户端先写一个父类的ArrayList程序

    public static void main(String[] args) throws Exception {
        List<String> list = new ArrayList<String>();
        for (int index = 0; index < 10; index++) {
            list.add(index + "");
        }

        List<String> subList = list.subList(1, 4);
        list.add(10 + "");
        System.out.println(subList);
    }

这个程序由于list.subList返回的List集合操作的还是原来list中的数据, 当我们调用原来数据的增删方法之后, 然后再调用subList那么会出现如下异常:

    Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$SubList.checkForComodification(ArrayList.java:1231)
	at java.util.ArrayList$SubList.listIterator(ArrayList.java:1091)
	at java.util.AbstractList.listIterator(AbstractList.java:299)
	at java.util.ArrayList$SubList.iterator(ArrayList.java:1087)
	at java.util.AbstractCollection.toString(AbstractCollection.java:454)
	at java.lang.String.valueOf(String.java:2982)
	at java.io.PrintStream.println(PrintStream.java:821)
	at com.terrylmay.springboot.Demo.main(Demo.java:25)

根据里式替换原则, 所有父类出现的地方都可以由子类透明的替换. 我们将程序替换之后看一下运行结果:

    public static void main(String[] args) throws Exception {
        List<String> list = new MyList<String>();
        for (int index = 0; index < 10; index++) {
            list.add(index + "");
        }

        List<String> subList = list.subList(1, 4);
        list.add(10 + "");
        System.out.println(subList);
    }

那么我们会发现, 程序已经不会报任何错误了.代码会输出:

    [1, 2, 3]

这样看来, 我们确实要避免重写父类中已经定义好的方法, 避免出现将父类对象替换为子类对象的时候出现行为不一致的情况, 违反里式替换原则事小, 如果因此程序出现不可预知的BUG就是大事了。

##### 2、子类可以拥有自己的方法

子类可以拥有自己的方法, 毕竟继承是为了重用代码. 而父类不可能全部实现所有的功能, 所以才有了继承的概念; 所以子类实现自己的方法也是非常正常的事情. 实现自己的方法最好是另外命名新的函数, 不要重新父类的函数;

比如上面的例子, 我们可以重构为如下代码而不违背里式替换原则:

    public static class MyList<T> extends ArrayList<T> {

        public List<T> subListWithoutException(int fromIndex, int toIndex) {
            ArrayList<T> subList = new ArrayList<T>(toIndex - fromIndex);
            for (int index = fromIndex; index < toIndex; index++) {
                subList.add(this.get(index));
            }

            return subList;
        }
    }

##### 3、当子类覆盖或实现父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松(本条含义经过测试发现已经不符合Java的语法了)

本来可以使用Map的例子来说明问题的, 但是貌似JDK1.8之后就已经不支持这种写法了

    public interface IPersonService {
      List<Person> getPersonList(HashMap params);
    }
    
    //该实现会报错, 说需要把类置为abstract或者实现List<Person> getPersonList(HashMap params);方法
    public class PersonService implements IPersonService {
        public List<Person> getPersonList(Map params) {
            return null;
        }
    }

##### 4、当子类覆盖或者实现父类的方法时, 方法的返回类型要比父类方法的返回值类型要严格

    public abstract class IPersonService {
      abstract List<Person> getPersonList(HashMap params);
    }

    public class PersonService extends IPersonService {
        public ArrayList<Person> getPersonList(HashMap params) {
            return null;
        }
    }

本来父类中使用了List<Person>作为返回值, 而在子类的实现中可以使用ArrayList<Person>进行返回, 这样当把IPersonService对象替换成PersonService对象的时候, 也不会发生异常情况

同时, 在Java中如果我们要实现Cloneable接口, 那么我们同样可以返回比Object范围小的数据类型

    public class Person implements Cloneable {
        @Override
        public Person clone() {
            return this;
        }
    }

本来Object中Clone的定义是需要返回Object类型的.

采用里式替换原则的目的就是为了增强程序的健壮性, 在版本升级过程中也可以保持非常好的兼容性. 在实际开发过程中, 我们也经常会用到接口 + 抽象骨干类 + 具体业务场景下的子类实现 组合来完成不同的业务逻辑, 而客户端调用的时候是使用接口或者父类作为引用的, 使用set方法传入接口或者父类的引用作为依赖注入的实现方式.



