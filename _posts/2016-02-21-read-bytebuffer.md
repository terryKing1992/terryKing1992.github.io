---
layout: post
title: ByteBuffer源码详解
date: 2016-02-21 08:41:12
tags: [ByteBuffer, Java Source]
---

上篇主要看到了关于buffer的结构的定义,搞来搞去还是那4个变量在起作用，所有的方法都是围绕了capacity>=limit>=position>=mark来起作用的。所以了解了这几个值的意义就相当于明白了buffer的工作原理了.这篇主要是看关于ByteBuffer的实现。

虽然本篇的题目是java IO探索，其实前面介绍了好多，除了FileInputStream之外，其他的好像都是NIO的内容,不过这些东西都是交叉，具有一定的耦合性，所以，某一篇涉及到了另外一个知识点就要把它搞清楚，这个类才算真正的搞清楚了。
NIO最主要的地方就是通过Channel和buffer对象来操作流数据，而ByteBuffer的内存分配不是在本地内存中分配内存，就是在jvm 堆中分配内存，在JVM中分配的内存受到垃圾回收机制的控制，而Direct ByteBuffer是通过JNI在Java虚拟机外的内存中分配了一块（所以即使在运行时通过-Xmx指定了Java虚拟机的最大堆内存，还是可能实例化超出该大小的Direct ByteBuffer），该内存块并不直接由Java虚拟机负责垃圾收集，但是在Direct ByteBuffer包装类被回收时，会通过Java Reference机制来释放该内存块这也就解释了为什么在BUffer类中有一个long型变量来存储地址了。这边应该是存储的JVM 分配的heap之外的内存，像c++一样访问。

下面还是看源码吧~

finalbyte[]hb; //这个是用来存储的缓冲字节的.其实说到底，所有的缓冲类肯定都是对字节或者其他数组的一种封装，本身类也是在基本类型上的一种扩展。


	ByteBuffer(intmark,int pos,intlim, intcap,  // package-private
					byte[] hb,intoffset) {

       super(mark,pos, lim, cap);

       this.hb = hb;

       this.offset =offset;

   }

对bytebuffer进行初始化，但是业务系统想要用这个方法是不可能的，因为它的属性是default的，表明包内可见。
offset这个变量暂时还没有发现它的用途是什么。到后面应该有解释吧。

	public static ByteBuffer allocateDirect(intcapacity) {
      return new DirectByteBuffer(capacity);
  	}

这个就是在JVM堆之外分配的直接内存DirectByteBuffer对象，下篇可以研究一下这个。

	publicstatic ByteBuffer allocate(intcapacity) {
	    if (capacity < 0)
	       throw new IllegalArgumentException();
	    return new HeapByteBuffer(capacity,capacity);
	}

分配堆内存。后天可以看一下堆内存对象HeapByteBuffer

	publicstatic ByteBuffer wrap(byte[]array,
	                               int offset, intlength)

	{
	    try {
	       return new HeapByteBuffer(array,offset,length);
	    } catch (IllegalArgumentException x) {
	       throw new IndexOutOfBoundsException();
	    }
	}

这个方法把array数组中的从offset开始起，复制length个字节到buffer中。

	publicabstract ByteBuffer slice();

返回一个新的Buffer对象，但是对象中的hp[]数组指向的地址与this对象是一个地址，就是两个ByteBuffer共享一个字节数组；

	publicabstract ByteBuffer duplicate();

这个方法跟slice()一样，不过一个是操作堆中的内存，一个操作直接内存。

	publicabstract ByteBuffer asReadOnlyBuffer();

返回一个只读的缓存，内容跟this中的值完全一样，两个buffer中的hp[]指向了同一块内存；

	publicabstractbyte get();//读模式

从buffer的position位置读取一个字节；

	publicabstract ByteBuffer put(byteb);//写模式

把一个字节放入到position之后，然后position的值+1；

	publicabstractbyte get(intindex);

读取buffer中index位置的字节，如果index超过了limit，或者小于position的值，则会抛出数组越界异常。

	publicabstract ByteBuffer put(intindex,byte b);

把字节写入到指定位置。index必须是合理的，并且这个buffer不是只读的。

	public ByteBuffer get(byte[]dst,int offset,intlength) {

	    checkBounds(offset,length, dst.length);

	    if (length > remaining())

	       throw new BufferUnderflowException();

	    int end = offset + length;

	    for (int i = offset; i < end; i++)

	       dst[i] = get();

	    return this;

	}

读取buffer中position开始的字节，然后放入到字节数组dst中，写入dst的时候从offset开始，读length到dst数组中。

	public ByteBuffer put(ByteBuffersrc) {
	    if (src == this)
	       throw new IllegalArgumentException();

	    if (isReadOnly())
	       throw new ReadOnlyBufferException();

	    int n = src.remaining();
	    if (n > remaining())
	       throw new BufferOverflowException();

	    for (int i = 0; i < n; i++)
	        put(src.get());

	    return this;
	}

从src这个buffer中的当前位置，读取buffer中未读字节到this buffer中。

	public ByteBuffer put(byte[]src,int offset,intlength) {

	    checkBounds(offset,length, src.length);
	    if (length > remaining())
	       throw new BufferOverflowException();

	    int end = offset + length;
	    for (int i = offset; i < end; i++)
	       this.put(src[i]);

	    return this;
	}

把src数组中的字节 从offset开始读取length字节到buffer中。如果要写入的数据length > buffer数组中未写入的数据(limit - position)，则会抛出异常.否则将数据写入到buffer中。

	publicfinal boolean hasArray() {
	    return (hb !=null) && !isReadOnly;
	}

判断这个buffer中是否有可读的数组，hb！=null 并且不是只读；

	publicfinal byte[] array() {
	    if (hb == null)
	       throw new UnsupportedOperationException();
	    if (isReadOnly)
	       throw new ReadOnlyBufferException();
	    return hb;
	}

返回该buffer中的存储的底层字节数组；

	public final int arrayOffset() {
	    if (hb == null)
	       throw new UnsupportedOperationException();
	    if (isReadOnly)
	       throw new ReadOnlyBufferException();
	    return offset;
	}

返回offset，但是还是没明白这个offset是干嘛用的，不然回来就得看别人写的到底是什么意思了。

这个类方法有点多了，明天再继续看这些方法吧.

虽然后面的这些方法都是抽象方法，不过知道这些方法的意思，对于后面的buffer子类的学习也是会有很大好处的.

	public abstract ByteBuffer compact();

截取this buffer中从position到limit得字节到另外一个buffer中，如果mark已经有值，那么设置mark=-1；设置返回的buffer的capacity=this.limit;

	public abstract boolean isDirect();

判断该buffer是否为直接内存buffer

	public int hashCode() {

	    int h = 1;
	    int p = position();
	    for (int i = limit() - 1; i >= p; i--)
	        h = 31 * h + (int)get(i);
	    return h;
	}

每一个类的hashcode都是算法的精髓，看怎么样设计hash能够造成对象最小的碰撞。这是需要考虑的，有时间可以把java中的所有对象的hash算法拿来观摩一下。
	 public boolean equals(Object ob) {

	    if (this == ob)
	        return true;

	    if (!(ob instanceof ByteBuffer))
	        return false;

	    ByteBuffer that = (ByteBuffer)ob;
	    if (this.remaining() != that.remaining())
	        return false;

	    int p = this.position();
	    for (int i = this.limit() - 1, j = that.limit() - 1; i >= p; i--, j--)
	        if (!equals(this.get(i), that.get(j)))
	            return false;

	    return true;
	}

判断两个buffer是否相等的判断条件，就是从position位置开始到limit位置结束，如果this buffer中的每个字节与另一个buffer中的从limit倒着数，直到this buffer的position，如果that中的字节与this完全相等，则说明两个buffer是相等的。

	public int compareTo(ByteBuffer that) {
	    int n = this.position() + Math.min(this.remaining(), that.remaining());
	    for (int i = this.position(), j = that.position(); i < n; i++, j++) {
	        int cmp = compare(this.get(i), that.get(j));
	        if (cmp != 0)
	            return cmp;
	    }
	    return this.remaining() - that.remaining();
	}

buffer既然实现了Compareable接口，那么必然要实现这个方法的。具体逻辑就自己看吧。

	public abstract CharBuffer asCharBuffer();

通过这个对象来得到一个charbuffer对象。

	public abstract ShortBuffer asShortBuffer();

通过这个对象得到shortbuffer对象

	public abstract short getShort(int index);

在index位置开始读取两个字节，并将这两个字节合并为一个short 类型变量

	public abstract int getInt();

从position位置读取4个字节数据，合并到一个int类型变量中去。

	public abstract IntBuffer asIntBuffer();

通过这个对象得到Intbuffer对象

	public abstract long getLong();

从position位置读取8个字节数据，合并到一个long类型变量中去。

	public abstract LongBuffer asLongBuffer();

后面肯定还会有floatbuffer以及doubleBuffer对象
基本上就是上面的这些方法了.回头要一个个实验一下到底是什么效果才记得更牢一点
