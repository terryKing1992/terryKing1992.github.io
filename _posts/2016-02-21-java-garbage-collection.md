---
layout: post
title: JVM垃圾回收机制中有趣的问题
date: 2016-02-21 10:37:46
tags: [JVM 内存回收]
---

大家都知道,垃圾回收机制是jvm在后台的不确定的某个时候开启垃圾回收线程来进行一些不可达对象的回收（已经没有指针指向该对象）.

基本上垃圾回收机制通常只回收堆以及本地方法栈里面的数据.对于方法区（perm space）或者叫它永久代里面的信息基本上不会回收..但也有例外的情况.因为回收这个里面的数据开销会比较大.回收回来的空间则相对于堆里面的要少了很多.话句话说就是性价比不高..好了,扯的远了.

<!-- more -->

本文的主题是关于垃圾回收一个有趣的问题的,下面就让我们看看这是一个什么问题

这个问题来自于《深入理解JVM》

对象的自我救赎：一个对象当没有被引用时,垃圾回收机制就会把该对象从堆中清除.那么,对象应该怎么自救呢?这就要说到在java中跟C++中的析构函数有点类似的finalize方法了.这个方法时超类Object中有的一个方法.这个方法一般人在做一般应用时是不会重载的.只有像书的作者想要告诉读者某件事情的时候会拿出来运行一下.不错.那么下面我们就从代码的角度看一下一个对象时怎么进行自我救赎的.

首先 ,在看代码之前,应该了解的一个知识点就是当垃圾回收器在进行垃圾回收某个对象的时候只会执行一次该对象的finalize方法.所以 在finalize方法还没有执行完的时候，再把该对象赋给某个全局的引用或者其他方法(只要有引用再次引用该对象的时候)该对象就成功逃过了被回收的命运

	public class Main {  
	    public static Main SAVE_HOOK = null;  

	    public void isAlive() {  
	        System.out.println("yes,i am still alive");  
	    }  

	    @Override  
	    protected void finalize() throws Throwable {  
	        // TODO Auto-generated method stub  
	        super.finalize();  
	        System.out.println("finalize method executed");  
	        Main.SAVE_HOOK = this;  
	    }  

	    /**
	     * @param args
	     */  
	    public static void main(String[] args) throws Exception {  
	        // TODO Auto-generated method stub  
	        SAVE_HOOK = new Main();  
	        SAVE_HOOK = null;  
	        System.gc();  
	        Thread.sleep(5000L);  
	        if (SAVE_HOOK != null) {  
	            SAVE_HOOK.isAlive();  
	        } else {  
	            System.out.println("i am die");  
	        }  

	        //  

	        SAVE_HOOK = null;  
	        System.gc();  
	        Thread.sleep(5000L);  
	        if (SAVE_HOOK != null) {  
	            SAVE_HOOK.isAlive();  
	        } else {  
	            System.out.println("i am die");  
	        }  
	    }  
	}

所以当第一次会成功摆脱被回收的命运、但是一个对象的finalize只会执行一次,当第二次再次回收的时候.那么该对象只能乖乖的被清除出heap中了.
执行程序就会看到输出为
其实我是非常赞同作者的观点的.finalize方法实现只是对C++的一种妥协.只是想告诉C++的一些程序员，java也是可以做到由程序员自己回收内存操作的.确实实现这个方法没有什么太大的用处.所以，还是非常不建议使用这个方法的..如果想要在最后清理掉某些外部资源的引用的话,finally代码块可以做的更好..
