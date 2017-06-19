---
title: Java中Buffer类源码学习
date: 2016-02-21 10:11:37
tags: [Buffer, 缓冲]
---

FileChannel类中直接通信的就是Buffer,Buffer是通道和文件之间的桥梁.

<!-- more -->

大概数据交互是这样的，FileChannel可以写数据到buffer中，同时另外一个输出FileChannel也可以从buffer中读取数据。
buffer中有两种模式：读模式和写模式，读模式会从position位置开始读取到limit字节；写模式同样也会从position写到limit，但是写模式中一般limit=capacity(buffer的大小)
Buffer 中有四个变量来标示一些位置：

	// Invariants: mark <= position <= limit <= capacity

	private int mark = -1;

	private int position = 0;

	private intlimit;

	private intcapacity;

这里面有一个恒不等式，就是上面指出的:mark<=position<=limit<=capacity

	 // Used only by direct buffers
	// NOTE: hoisted here for speed in JNI GetDirectBufferAddress
	longaddress;

有一个存储关于地址的变量.看解释好像是为了提高JNI的速度。

下面是Buffer得构造函数，声明为default类型的，只在包里面可见

	Buffer(int mark, int pos, int lim, int cap) {       // package-private
	    if (cap < 0)
	        throw new IllegalArgumentException("Negative capacity: " +cap);

	    this.capacity =cap;
	    limit(lim);
	    position(pos);
	    if (mark >= 0) {
	        if (mark >pos)
	            throw new IllegalArgumentException("mark > position: ("
                         + mark + " > " + pos + ")");
	        this.mark =mark;
	    }
	}

下面是不允许重写的capacity()和position()方法：

	public final int capacity() {
	    return capacity;
	}
	public final int position() {
	    return position;
	}

设置position的值，以便能够满足重复读取通道中已经读过的内容需求，如果设置position时，使得mark的值>position,那么mark的值将会置为-1；下面是这个方法的实现：

	public final Buffer position(intnewPosition) {
	    if ((newPosition >limit) || (newPosition < 0))
	        throw new IllegalArgumentException();
	    position = newPosition;
	    if (mark > position) mark = -1;
	    return this;
	}

设置limit的值，如果limit>capacity那么抛出不合法参数异常，否则设置limit = newLimit,如果当前buffer的position的值大于limit，那么设置position=limit，要不然在写数据的时候从position读到limit会永远都写不进去了，可能会出现数据不一致或者读写数据出错的情况，这点要注意；

	public final Buffer limit(intnewLimit) {
	    if ((newLimit >capacity) || (newLimit < 0))
	        throw new IllegalArgumentException();
	    limit = newLimit;
	    if (position >limit) position = limit;
	    if (mark > limit) mark = -1;
	    return this;
	}

如果buffer调用了mark()方法，那么会使得mark的值等于position，每次读数据或者写数据的时候position会递增，设置，调用mark方法，应该可以从下次再重复读取内容了。但是在buffer中没有找到reset方法，所以目前这个作用也只是个猜测(卧槽，这不坑爹么.刚才猜测的时候全局搜索reset根本找不到.竟然在mark方法的下一个方法就是reset，这就显而易见了，position都回到了mark地方了，肯定就可以重复读取了.)

	public final Buffer reset() {
	    int m = mark;
	    if (m < 0)
	        throw new InvalidMarkException();
	    position = m;
	    return this;
	}

调用clear方法会使得数据全部回归原来的状态，这样在写入buffer的时候，就又可以从position=0开始写入了，如果position = limit的话，就没有办法写入到buffer中了。

	public final Buffer clear() {
	    position = 0;
	    limit = capacity;
	    mark = -1;
	    return this;
	}

但是buffer中的内容根本不会清除，如果在clear()之后又去读里面的内容很可能就会造成问题，刚才试了试，会造成死循环：代码如下：
	
	//读数据到buffer中，比如position=0,limit=capacity=50,输入通道中有15个字节
	while(channelIn.read(buffer) != -1) {
	//这样的话，buffer中的值应该是position=15，limit=capacity=50，mark=-1；调用flip()方法之后
	buffer.flip();
	//会使得limit= position =15，position = 0；capacity = 50，mark = -1；
	channelOut.write(buffer);
	//将buffer中的数据读入到输出channelOut中，这时数据为position=15，limit=capacity=50，mark=-1；
	buffer.clear();
	//调用buffer.clear()之后，使得limit=position = 15，position=0,capacity = limit = 50,mark = -1
	channelOut.write(buffer);

	//使用write的时候操作buffer,是从position到limit的数据全部读入到输出通道中。所以，buffer中的值变为：position=limit = capacity =50，那么因为position=limit所以下次再读数据进buffer得时候，因为缓冲区是满的，所以无法读数据到缓冲区，造成了读入字节数为0，这样一直循环，就会一直把数据写入到文件中，并且造成文件空洞(文件中的数据之间有空隙)。而使得数据异常(不一致)。

当然flip在上篇文章中就介绍过.不过好像没有说道具体实现，现在看一下具体实现，其实就是上篇说到的怎么切换到读模式的

	public final Buffer flip() {
	    limit = position;
	    position = 0;
	    mark = -1;
	    return this;
	}

下面这个方法，设置position=0，使得数据能够重写写入并且覆盖原来的写入，也可以使得读取的时候从头开始重复读取。
 
	public final Buffer rewind() {
	    position = 0;
	    mark = -1;
	    return this;
	}

下面这个函数有两层含义：一个是当读取的时候，表示还有多少字节没有读过；写模式的时候表示还剩余多少空间可写。

	public final int remaining() {
	    return limit -position;
	}

获取buffer的可读属性，判断是否为只读

	public abstract boolean isReadOnly();

设置当前位置之后的多少位：功能跟position(int n)一样

	final int nextGetIndex() {                         // package-private
	    if (position >=limit)
	        throw new BufferUnderflowException();
	    return position++;
	}
	final int nextGetIndex(intnb) { // package-private
	    if (limit -position < nb)
	        throw new BufferUnderflowException();
	    int p = position;
	    position += nb;
	    return p;
	}

下面这两个函数跟上面的完全一样，就是逻辑上一个是读一个是写

	final int nextPutIndex() {                         // package-private
	    if (position >=limit)
	        throw new BufferOverflowException();
	    return position++;
	}

	final int nextPutIndex(intnb) {                    // package-private
	    if (limit -position < nb)
	        throw new BufferOverflowException();
	    int p = position;
	    position += nb;
	    return p;
	}

检测从i开始的第nb个位置的数据是否合法

	final int checkIndex(inti, int nb) {              // package-private
	    if ((i < 0) || (nb >limit - i))
	        throw new IndexOutOfBoundsException();

	    return i;
	}

截断缓冲区，并且调用这个方法后，这个缓冲区就会变的不可用

	final void truncate() {                            // package-private
	    mark = -1;
	    position = 0;
	    limit = 0;
	    capacity = 0;
	}

总得来说，buffer的这些方法都是围绕了buffer得对象模型展开的，主要操作元素就是对象中的mark，position，limit以及capacity属性。理解了这层含义，那么对于buffer的操作便会顺心应手。