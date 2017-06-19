---
title: Java中ArrayList类源码学习
date: 2016-02-21 10:22:02
tags: [ArrayList, 容器]
---

相信大家对ArrayList都是比较熟悉的.ArrayList也是在软件开发中经常使用到的一个集合，相对于Vector集合来说，ArrayList的每一个方法少了一个修饰符那就是synchronized，也就意味着该集合类不是线程安全的.而Vector类是线程安全的.但有一点没有想通的是人们更加倾向于使用ArrayList，并且在线程中使用锁机制实现同步，也不使用Vector类来实现同步；好了废话不多说了.

<!-- more -->

下面进入ArrayList的学习

首先，打开java.util包下面的ArrayList.java我们可以看到类名以及该类的继承关系

	public class ArrayList<E> extends AbstractList<E>  
	    implements List<E>, RandomAccess, Cloneable, java.io.Serializable  

除了类名不一样外，其他的根Vector类的继承关系一模一样。这里就不在过多的描述了。往下看，它里面的成员变量

	private static final int DEFAULT_CAPACITY = 10;  
	private static final Object[] EMPTY_ELEMENTDATA = {};  
	private transient Object[] elementData;  
	private int size;  

虽然在Vector中没有提到过它里面的变量，但是从这四行代码中可以看到ArrayList底层也是通过数组的形式实现的，第一行中的默认大小为10；以及第二行中定义了一个空的数组，而且是final类型的.这个就十分没有必要了。这个空数组虽然在后面的无参构造函数中赋值给了elementData，但是这种做法一点意义都没有.

第三行代码定义了存储数据的数组，但是它的修饰符是transient，这个修饰符的意思是在该对象进行序列化的时候，不对该数组进行序列化.

而一般在进行数据传输时才会用到序列化。这样修饰感觉对序列化完全没有了意义。int size ；从这行代码就可以看出来，显然这个类的实现作者跟Vector类的实现作者不是同一个人了。一个使用了变量protected int elementCount;另一个使用了变量private int size;不过从后面的代码可以看出Vector的实现作者显然能力更强一些，也更加机智一些。下面接着跟着代码走
	
	public ArrayList() {  
	       super();  
	       this.elementData = EMPTY_ELEMENTDATA;  
	}  

上面提到的EMPTY_ELEMENTDATA数组变量的唯一用武之地，但是这样的无参构造有必要么?显然。他要是这么定义我觉得还更合理一些

	public ArrayList() {  
	        super();  
	        this(DEFAULT_CAPACITY);  
	} 

看过Vector篇的应该知道Vector中扩容的时候会有一个扩容基数capacityIncrement，而ArrayList中的扩容基数为扩容前的数组大小的1.5倍，代码中可见：
	
	private void grow(int minCapacity) {  
	    // overflow-conscious code  
	    int oldCapacity = elementData.length;  
	    //设置新的容量大小  
	    int newCapacity = oldCapacity + (oldCapacity >> 1);  
	    if (newCapacity - minCapacity < 0)  
	        newCapacity = minCapacity;  
	    if (newCapacity - MAX_ARRAY_SIZE > 0)  
	        newCapacity = hugeCapacity(minCapacity);  
	    // minCapacity is usually close to size, so this is a win:  
	    elementData = Arrays.copyOf(elementData, newCapacity);  
	}  

其他方法如remove（），indexOf等方法跟Vector中的一模一样，全部是对数组的操作.这里就不再多说了,这个类中一个矛盾的地方还是在序列化这个地方，要深入专研一下

先看一下代码中关于序列化的实现

	private void writeObject(java.io.ObjectOutputStream s)  
	   throws java.io.IOException{  
	   // Write out element count, and any hidden stuff  
	   int expectedModCount = modCount;  
	   s.defaultWriteObject();  

	   // Write out size as capacity for behavioural compatibility with clone()  
	   s.writeInt(size);  

	   // Write out all elements in the proper order.  
	   for (int i=0; i<size; i++) {  
	       s.writeObject(elementData[i]);  
	   }  

	   if (modCount != expectedModCount) {  
	       throw new ConcurrentModificationException();  
	   }  
	}

由代码可以看出，虽然在elementData前面加上了transient修饰符，但是在序列化对象的时候该类又尝试着把所有的元素都序列化输出了。以前序列化输出并不常见，但是现在的确需要好好研究一下序列化的问题了。

首先，先看一下ArrayList对象在另一个对象中能否序列化输出，下面是写的代码进行测试

	public class MyExam12_1 implements Serializable {  
	    public ArrayList<Integer> array = new ArrayList<Integer>();  

	    public MyExam12_1() {  
	        array.add(1);  
	        array.add(1);  
	        array.add(1);  
	        array.add(1);  
	    }  

	    public static void main(String[] args) {  
	        MyExam12_1 cat = new MyExam12_1();  
	        try {  
	            FileOutputStream fos = new FileOutputStream("catDemo.out");  
	            ObjectOutputStream oos = new ObjectOutputStream(fos);  
	            System.out.println(" 1> " + Arrays.toString(cat.array.toArray()));  
	            oos.writeObject(cat);  
	            oos.close();  
	        } catch (Exception ex) {  
	            ex.printStackTrace();  
	        }  
	        try {  
	            FileInputStream fis = new FileInputStream("catDemo.out");  
	            ObjectInputStream ois = new ObjectInputStream(fis);  
	            cat = (MyExam12_1) ois.readObject();  
	            System.out.println(" 2> " + Arrays.toString(cat.array.toArray()));  
	            ois.close();  
	        } catch (Exception ex) {  
	            ex.printStackTrace();  
	        }  
	    }  
	}  

通过将MyExam12_1对象cat进行序列化，放入到对象输出流中，然后通过再把信息放入到对象输入流中，看到是可以将ArrayList中的数据元素输出的。输出结果如下：

但是如果使用另外一个代码进行测试，将ArrayList变成数组的形式，并且使用transient修饰该数组，进行测试，代码如下：

	public class MyExam12_1 implements Serializable {  
	    public transient int[] array = { 1, 2, 3, 4, 5, 6, 7 };  

	    public MyExam12_1() {  
	    }  

	    public static void main(String[] args) {  
	        MyExam12_1 cat = new MyExam12_1();  
	        try {  
	            FileOutputStream fos = new FileOutputStream("catDemo.out");  
	            ObjectOutputStream oos = new ObjectOutputStream(fos);  
	            System.out.println(" 1> " + Arrays.toString(cat.array));  
	            oos.writeObject(cat);  
	            oos.close();  
	        } catch (Exception ex) {  
	            ex.printStackTrace();  
	        }  
	        try {  
	            FileInputStream fis = new FileInputStream("catDemo.out");  
	            ObjectInputStream ois = new ObjectInputStream(fis);  
	            cat = (MyExam12_1) ois.readObject();  
	            System.out.println(" 2> " + Arrays.toString(cat.array));  
	            ois.close();  
	        } catch (Exception ex) {  
	            ex.printStackTrace();  
	        }  
	    }  
	} 

通过测试发现，无法正常进行序列化输出，结果如下：

如果是这样的话，可以猜测，ArrayList类中是重写了readObject方法以及writeObject方法才使得其对象elementData中的元素能够正常序列化的,现在我们模仿ArrayList中的方法，对上述代码进行改进，看看能不能得到序列化数组

	public class MyExam12_1 implements Serializable {  
	    public transient Object[] array = { 1, 2, 3, 4, 5, 6, 7 };  

	    public MyExam12_1() {  
	    }  

	    private void writeObject(java.io.ObjectOutputStream s)  
	            throws java.io.IOException {  
	        // Write out element count, and any hidden stuff  
	        s.defaultWriteObject();  

	        // Write out all elements in the proper order.  
	        for (int i = 0; i < 7; i++) {  
	            s.writeObject(array[i]);  
	        }  
	    }  

	    private void readObject(java.io.ObjectInputStream s)  
	            throws java.io.IOException, ClassNotFoundException {  
	        // Read in size, and any hidden stuff  
	        s.defaultReadObject();  
	        array = new Object[7];  
	        Object[] object = array;  
	        // Read in all elements in the proper order.  
	        for (int i = 0; i < 7; i++) {  
	            object[i] = s.readObject();  
	        }  
	    }  
	}

并且在main方法中进行测试

	public class Main {  
	    public static void main(String[] args) {  
	        MyExam12_1 cat = new MyExam12_1();  
	        try {  
	            FileOutputStream fos = new FileOutputStream("catDemo.out");  
	            ObjectOutputStream oos = new ObjectOutputStream(fos);  
	            System.out.println(" 1> " + Arrays.toString(cat.array));  
	            oos.writeObject(cat);  
	            oos.close();  
	        } catch (Exception ex) {  
	            ex.printStackTrace();  
	        }  
	        try {  
	            FileInputStream fis = new FileInputStream("catDemo.out");  
	            ObjectInputStream ois = new ObjectInputStream(fis);  
	            cat = (MyExam12_1) ois.readObject();  
	            System.out.println(" 2> " + Arrays.toString(cat.array));  
	            ois.close();  
	        } catch (Exception ex) {  
	            ex.printStackTrace();  
	        }  
		}  
	}

这样就可以了.所以还是readObject以及writeObject函数在起作用，但是ArrayList中的elementData完全没有必要将它定义为transient类型的..感觉这个类中很多不合理的地方.