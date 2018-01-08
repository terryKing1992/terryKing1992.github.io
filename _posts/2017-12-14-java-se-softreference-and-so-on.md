---
layout: post
title: Java中强引用、软引用、弱引用、虚引用的区别与用法
date: 2017-12-14 21:30:00
tags: [软引用, 弱引用, 虚引用, 内存管理]
---

在Java中提供了对象的4个级别引用, 分别是强引用、软引用、弱引用以及虚引用。这四个类型的引用中, 只有强类型的引用是包内可见的, 其他级别的引用都是Public, 可以直接被应用程序开发者使用的。

### 强引用

我们在Java中创建的对象引用, 一般都是强类型引用. 这同样意味着当该对象如果有引用存在的情况下, 同时在JVM整个环境中该对象是路径可达的, 那么该对象永远不会被垃圾回收机制回收的。
例如我们创建一个对象```Object object = new Object();``` 该object就是一个强引用. 当强引用的对象占用内存过多, 而又没有释放的时候, 就会出现OOM问题; 下面我们看一个比较极端的例子:

```java
    public static void main(String[] args) {
        List<Byte[]> list = new ArrayList<Byte[]>();
        int index = 0;
        while (index++ < 1000000) {
            Byte[] bytes = new Byte[1024 * 1024];
            list.add(bytes);
        }
    }
```

当运行程序过段时间, 我们就能够看到如下打印信息:

```java
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.terrylmay.springboot.Demo.main(Demo.java:13)
```

表示我们的程序占用内存过多, 而对象的强引用还存在 无法回收对象, 从而造成堆内存无可用空间分配新的内存。在强引用情况下, JVM宁可出现OOM异常也不会回收强引用还存在的对象 以避免造成程序的一些意外情况。

### 软引用

软引用使用的时候需要通过一个软引用对象来进行声明, 软引用对象比强引用稍微弱一点, 通过软引用的对象当出现内存不足的情况的时候, 垃圾回收机制会将该类型的对象进行回收;我们通过如下代码创建一个弱引用的对象

```java
    String object = new String("HelloWorld");
    SoftReference<String> softRef = new SoftReference<String>(object)
```

既然软引用在内存不足的情况下会回收内存, 那么我们改写上面的程序看是否还会出现OOM问题

```java
    public static void main(String[] args) {
        List<SoftReference<Byte[]>> list = new ArrayList<SoftReference<Byte[]>>();
        int index = 0;
        while (index++ < 2000) {
            Byte[] bytes = new Byte[1024 * 1024];
            SoftReference<Byte[]> softReference = new SoftReference<Byte[]>(bytes);
            bytes = null;
            list.add(softReference);
        }
        SoftReference<Byte[]> byteFirst = list.get(0);
        System.out.println(byteFirst.get() == null);
    }   
```

通过运行上面代码我们可以发现, 程序并不会出现OOM的情况, 同时当程序运行完成之后, 打印list的第一个元素的reference对象(Byte[]数组), 发现程序会打印如下信息:

    true

表明bytes引用的对象已经被释放掉了。

### 弱引用

弱引用使用的时候需要通过一个弱引用对象进行声明。弱引用可以维持对对象的引用, 但是一旦垃圾回收线程工作的时候, 发现一个对象只有弱引用保持的时候, 那么就会对该对象进行垃圾回收.引用类型是一种比软引用弱的易用类型, 在系统GC时, 只要发现一个对象只有弱引用时不管系统堆空间是否足够, 都会将对象回收。但是由于垃圾回收器的线程通常优先级很低, 因此并不一定能够很快发现只有弱引用持有的对象, 在这种情况下, 弱引用对象可以存在很长的时间。一旦弱引用对象被垃圾回收器回收, 那么有一种机制能够保证该弱引用添加到一个注册引用的队列中。

下面我们看一个使用弱引用并且当系统 GC的时候发生的情况(当内存不足的情况下发生GC)

```java
    public static void main(String[] args) throws Exception {
        final List<WeakReference<Byte[]>> list = new ArrayList<WeakReference<Byte[]>>();
        int index = 0;
        final ReferenceQueue<Byte[]> queue = new ReferenceQueue<Byte[]>();

        while (index++ < 2000) {
            Byte[] bytes = new Byte[1024 * 1024];
            WeakReference<Byte[]> weakReference = new WeakReference<Byte[]>(bytes, queue);
            bytes = null;
            list.add(weakReference);
        }

        Thread thread = new Thread(new Runnable() {
            public void run() {
                try {
                    Reference reference = queue.remove();
                    while(reference!=null) {
                        System.out.println("当前的Byte[]对象引用地址为:" + reference);
                        System.out.println("当前加入到引用队列的Byte[]对象引用存在于List中:" + list.contains(reference));
                        System.out.println("当前Byte[]对象引用所在List的位置为:" + list.indexOf(reference));
                        Byte[] referent = (Byte[])reference.get();
                        //这一句话打印的非常奇怪, 竟然返回的是false, 但是下面一句打印地址的时候又打印出了null
                        System.out.println("当前Byte[]对象引用指向的对象是否为空" + referent == null);
                        System.out.println("当前Byte[]对象引用指向的对象是否为null字符串" + "null".equals(referent));

                        System.out.println("当前Byte[]对象地址为:" + referent);
                        reference = queue.remove();
                    }

                }catch (InterruptedException e) {
                    System.out.println(e.getCause());
                }

            }
        });

        thread.setDaemon(true);
        thread.start();
        Thread.sleep(100000);
    }
```

我们从输出可以看到日志打印为:

```java
    当前的Byte[]对象引用地址为:java.lang.ref.WeakReference@3f6e8054
    当前加入到引用队列的Byte[]对象引用存在于List中:true
    当前Byte[]对象引用所在List的位置为:1925
    false
    当前Byte[]对象引用指向的对象是否为null字符串false
    当前Byte[]对象地址为:null

    当前的Byte[]对象引用地址为:java.lang.ref.WeakReference@31a979c9
    当前加入到引用队列的Byte[]对象引用存在于List中:true
    当前Byte[]对象引用所在List的位置为:1924
    false
    当前Byte[]对象引用指向的对象是否为null字符串false
    当前Byte[]对象地址为:null
```

由于日志较长, 我们可以看到日志中值释放了1925索引位置及以前的Byte[]数据, 而1925以后的并没有释放. 这说明GC发生在创建第1926个对象之前, 所以JVM在创建后面75个对象的时候并没有出现内存不足的情况导致的GC发生, 所以后面75个对象应该都还能获取到Byte[]的对象地址.

因为对于弱引用来说, 任何时候只要系统发生GC都会引发WeakReference引用对象的释放. 我们把程序中由内存不足引起的GC换成手工调用GC方法

```java
    public static void main(String[] args) throws Exception {
            final List<WeakReference<Byte[]>> list = new ArrayList<WeakReference<Byte[]>>();
            int index = 0;
            final ReferenceQueue<Byte[]> queue = new ReferenceQueue<Byte[]>();

            while (index++ < 2) {
                Byte[] bytes = new Byte[1024 * 1024];
                WeakReference<Byte[]> weakReference = new WeakReference<Byte[]>(bytes, queue);
                bytes = null;
                list.add(weakReference);
            }
            System.gc();
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    try {
                        Reference reference = queue.remove();
                        while(reference!=null) {
                            System.out.println("当前的Byte[]对象引用地址为:" + reference);
                            System.out.println("当前加入到引用队列的Byte[]对象引用存在于List中:" + list.contains(reference));
                            System.out.println("当前Byte[]对象引用所在List的位置为:" + list.indexOf(reference));
                            Byte[] referent = (Byte[])reference.get();
                            //这一句话打印的非常奇怪, 竟然返回的是false, 但是下面一句打印地址的时候又打印出了null
                            System.out.println("当前Byte[]对象引用指向的对象是否为空" + referent == null);
                            System.out.println("当前Byte[]对象引用指向的对象是否为null字符串" + "null".equals(referent));

                            System.out.println("当前Byte[]对象地址为:" + referent);
                            reference = queue.remove();
                        }

                    }catch (InterruptedException e) {
                        System.out.println(e.getCause());
                    }

                }
            });

            thread.setDaemon(true);
            thread.start();
            Thread.sleep(100000);
    }
```
    
可以发现日志也打印出了对象回收的相关信息

```java
    当前的Byte[]对象引用地址为:java.lang.ref.WeakReference@10dd4940
    当前加入到引用队列的Byte[]对象引用存在于List中:true
    当前Byte[]对象引用所在List的位置为:1
    false
    当前Byte[]对象引用指向的对象是否为null字符串false
    当前Byte[]对象地址为:null
    当前的Byte[]对象引用地址为:java.lang.ref.WeakReference@1e1aa52b
    当前加入到引用队列的Byte[]对象引用存在于List中:true
    当前Byte[]对象引用所在List的位置为:0
    false
    当前Byte[]对象引用指向的对象是否为null字符串false
    当前Byte[]对象地址为:null
```

上面例子更加说明了弱引用当GC发生的时候就会回收相关的对象。


### 虚引用

虚引用与其他的引用都不相同, 虚引用并不会决定引用对象的生命周期, 所以虚引用又成为"幽灵引用"。如果一个对象只持有虚引用, 那么该对象就跟没有任何引用一样, 在任何时候都有可能被垃圾回收器回收. 所以没办法在程序中调用它的任何相关函数, 因为存在太多的不确定性。而虚引用主要是用来提供当对象被GC的时候的通知机制. 通过该通知机制我们可以做一些资源回收等方面的工作. 因为对象只存在虚引用完全没有意义, 即在程序中声明一个对象的虚引用完全没有意义, 所以虚引用一定要与Reference队列一起使用, 才能起到对象回收通知机制.

虚引用与引用队列的使用方法基本上面提到的软应用的使用方法一致. 这里就不具体贴相关使用代码了.

### 软引用、弱引用以及虚引用的finalize与进队列的时机

在<<大话Java性能优化>>这本书的第3.4.5节:"当对象引用进入到引用队列之后, 对象的finalize方法可能还没有被调用", 我针对该说法做了如下代码的实验:

```java
    package com.terrylmay.springboot;

    import java.lang.ref.Reference;
    import java.lang.ref.ReferenceQueue;
    import java.lang.ref.WeakReference;
    import java.util.ArrayList;
    import java.util.List;

    public class Demo {

        public static void main(String[] args) throws Exception {
            final List<WeakReference<ReferenceObject>> list = new ArrayList<WeakReference<ReferenceObject>>();
            int index = 0;
            final ReferenceQueue<ReferenceObject> queue = new ReferenceQueue<ReferenceObject>();

            while (index++ < 2) {
                ReferenceObject referenceObject = new ReferenceObject(index + "");
                WeakReference<ReferenceObject> weakReference = new WeakReference<ReferenceObject>(referenceObject, queue);
                referenceObject = null;
                list.add(weakReference);
            }

            System.gc();

            Thread thread = new Thread(new Runnable() {
                public void run() {
                    try {
                        Reference reference = queue.remove();
                        while(reference!=null) {
                            System.out.println("当前的Byte[]对象引用地址为:" + reference);
                            System.out.println("当前加入到引用队列的Byte[]对象引用存在于List中:" + list.contains(reference));
                            System.out.println("当前Byte[]对象引用所在List的位置为:" + list.indexOf(reference));
                            Byte[] referent = (Byte[])reference.get();
                            //这一句话打印的非常奇怪, 竟然跟
                            System.out.println("当前Byte[]对象引用指向的对象是否为空" + referent != null);
                            System.out.println("当前Byte[]对象引用指向的对象是否为null字符串" + "null".equals(referent));

                            System.out.println("当前Byte[]对象地址为:" + referent);
                            reference = queue.remove();
                        }
                    }catch (InterruptedException e) {
                        System.out.println(e.getCause());
                    }

                }
            });

            thread.setPriority(Thread.MAX_PRIORITY);
            thread.setDaemon(true);
            thread.start();
            Thread.sleep(100000);
        }

        public static class ReferenceObject {
            private String name;
            private Byte[] bytes;

            public ReferenceObject(String name) {
                this.name = name;
                this.bytes = new Byte[1024 * 1024];
            }

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public Byte[] getBytes() {
                return bytes;
            }

            public void setBytes(Byte[] bytes) {
                this.bytes = bytes;
            }

            @Override
            protected void finalize() throws Throwable {
            System.out.println("系统调用finalize方法回收了name=" + this.name + " 的对象");
            }
        }
    }
```

发现每次都是先调用ReferenceObject的finalize方法, 然后才会进入到队列中的。由于我启动的线程 与 ReferenceQueue入队列的线程都具有最高优先级, 而垃圾回收线程的优先级相对很低, 而且我尝试运行了很多次代码, 都会打印出相同的结果如下:

```java
    系统调用finalize方法回收了name=2 的对象
    系统调用finalize方法回收了name=1 的对象
    当前的Byte[]对象引用地址为:java.lang.ref.WeakReference@22ac81b3
    当前加入到引用队列的Byte[]对象引用存在于List中:true
    当前Byte[]对象引用所在List的位置为:1
    true
    当前Byte[]对象引用指向的对象是否为null字符串false
    当前Byte[]对象地址为:null
    当前的Byte[]对象引用地址为:java.lang.ref.WeakReference@2c6b7b96
    当前加入到引用队列的Byte[]对象引用存在于List中:true
    当前Byte[]对象引用所在List的位置为:0
    true
    当前Byte[]对象引用指向的对象是否为null字符串false
    当前Byte[]对象地址为:null
```

所以我推测, 只有当gc方法调用了finalize之后, 才会进入到ReferenceQueue队列中, 实验结果与书中描述不符. 更加严谨的证据还需要在JVM代码中查看.等后续查看到相关代码再进行补充。




