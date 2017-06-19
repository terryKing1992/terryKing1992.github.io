---
layout: post
title: 线性表的数组实现
date: 2016-2-21 10:54:00
tags:
---

线性表数组实现代码
线性表 的数组实现:线性表中应该有的功能包括
增加删除查找以及修改的操作以及对象的初始化，自增操作以及返回线性表的数据元素的个数

	package com.neusoft.linearlist;


	import java.util.Arrays;

	public class ArrayList<T> {

		//线性表的容量
		private int capacity;
		// 线性表中元素的个数
		private int size;
		// 存储数据的数组
		private Object[] elements;
		// 初始容量
		private final static int initCapacity = 16;


		// 无参构造
		public ArrayList() {
		    capacity = initCapacity;
		    elements = new Object[capacity];
		    size = 0;
		}


		/*
		* @param initCapacity
		*/
		// 初始容量
		public ArrayList(int initCapacity) {
		    capacity = initCapacity;
		    elements = new Object[capacity];
		    size = 0;
		}


		//返回元素个数
		public int size() {
		    return size;
		}


		// ·返回索引出元素
		/*
		* @param index
		*/
		@SuppressWarnings("unchecked")
		public T get(int index) {
		    if (index < 0 || index > size) {
		    throw new ArrayIndexOutOfBoundsException();
		    }
		    return (T) elements[index];
		}


		/*
		* @function insert
		*
		* @param index element
		*/
		public void insert(int index, T element) {
		    if (index < 0 || index > size) {
		    throw new ArrayIndexOutOfBoundsException();
		    }
		    ensureCapacity(size + 1);
		    System.arraycopy(elements, index, elements, index + 1, size - index);
		    elements[index] = element;
		    size++;
		}


		/*
		* 容量自增
		*
		* @param int size
		*/
		private void ensureCapacity(int size) {
		    if (size > capacity) {
		        while (capacity < size)
		        capacity <<= 1;
		    elements = Arrays.copyOf(elements, capacity);
		    }
		}


		/*
		* 删除指定索引的元素
		*
		* @param index
		*/
		@SuppressWarnings("unchecked")
		public T remove(int index) {
		    if (index < 0 || index >= size) {
		    throw new ArrayIndexOutOfBoundsException();
		    }
		    Object oldElement = elements[index];
		    System.arraycopy(elements, index + 1, elements, index, size - index - 1);
		    elements[--size] = null;
		    return (T) oldElement;
		}


		/*
		* Ï增加元素
		*
		* @param element
		*/
		public void add(T element) {
		    this.insert(size, element);
		}


		/*
		* 删除最后元素
		*/
		public T remove() {
		    return this.remove(size - 1);
		}

		/*
		* ·判断是否为空
		*/
		public boolean isEmpty() {
		    return size == 0;
		}
	}

还是想说的一件事在用数组实现线性表的时候学过数据结构的都应该知道
在数组实现的线性表中当对线性表进行在里面的插入和删除的时候会有数组中元素的逐个移动.导致这样的数据结构在插入和删除的时候效率很低..
同样要自己实现关于数据的移位删除操作比如

	for(int i=index;i<size-1;i++){
	   array[i]=array[i+1];
	}
	size--;

不知道有人想过没有如果这样做的话确实可以实现index后面的数据全部都向前面进行移位但是原来在索引为size位置上的数据其实本来已经没用了..

到后面肯定还会被其他数据覆盖的.但是程序中并没有释放array[size]的空间.因为JVM只有在某个对象不可达的情况下才会释放对象的空间..但是array[size-1]该对象还是可达的.因为array[size]还是可以访问到堆中这个对象的..所以JVM不会马上就释放.虽然可能自己写的程序并不允许调用者访问array[size]，因为这样的话就会出现IndexOutOfBoundsException异常..不过程序内部还是可以访问的..不过JVM只看该对象是否可达为根据来释放空间的..

所以我们也要把该对象的引用释放及在程序的末尾加上这么一段代码array[size]=null；

这样才能保证内存不被浪费和合理的运用

除了自己写方法来完成数据的移位操作外系统中还有关于数组的拷贝操作..

同样jdk中有一个系统类System中提供System.arraycopy(array,index+1,array,index,size-index-1);

参数说明:第一个参数:源数组，第二个参数:源数组开始拷贝的起始位置 ，第三个参数为目标数组，第四个参数为目标数组的从第几个位置开始 ，第五个参数为要拷贝的元素的个数

这样就能完成数组数据的移位了..

还有一个Arrays工具类来完成数组对象的拷贝.这样的方法是全部拷贝.且返回一个数组类型

	array=Arrays.copyOf(array,capacity);

这样就会把array中的数组元素拷贝到大小为capacity的数组中..并且把array指向刚分配的数组空间
