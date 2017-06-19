---
title: Java中FileChannel源码学习
date: 2016-02-21 09:30:42
tags: [FileChannel]
---

昨天看到了FileInputStream类的内部实现，里面除了继承自InputStream类外，还多加了一个FileChannel的方法，通过getChannel()方法获取到文件的通道，今天看一下关于这个FileChannel到底做什么操作，它比FileInputStream有什么强大的地方?以及它的内部实现具体是怎样的。

<!-- more -->

下面就看看FileChannel具体类实现是什么样的:

刚开始就能看到相关类的介绍：

	A channel for reading, writing, mapping, and manipulating a file.

对文件进行读写，映射以及操作的通道。

类定义如下：
	
	public abstractclass FileChannel
			extends AbstractInterruptibleChannel
			implements SeekableByteChannel, GatheringByteChannel, ScatteringByteChannel

实现了好多接口啊..不过这些接口以后肯定会碰到并且了解其主要功能的。本篇主要学习FileChannel类

	protected FileChannel() { }

受保护的构造方法，只能够被自己和其子类访问。下面这个方法是jdk1.7提供的：

	public static FileChannel open(Path path,
        Set<?extends OpenOption>options,
				 FileAttribute<?>...attrs) throws IOException {

	    FileSystemProviderprovider = path.getFileSystem().provider();

	   	return provider.newFileChannel(path,options,attrs);
	}

通过路径Path类来获取FileSystemProvider 并且创建一个FileChannel对象来操作文件

	private static final FileAttribute<?>[] NO_ATTRIBUTES =new FileAttribute[0];

提供默认文件属性:

	publics tatic FileChannel open(Path path, OpenOption...options)
								throws IOException {
		Set<OpenOption>set = new HashSet<OpenOption>(options.length);
    	Collections.addAll(set,options);
		return open(path,set,NO_ATTRIBUTES);
	}

同样是类函数，返回FileChannel对象

	/**

	 * Reads a sequence of bytes from this channel into the given buffer.

	 *

	 *<p> Bytes are read starting at this channel's current file position, and

	 * then the file position is updated with the number of bytes actually

	 * read.  Otherwise this method behaves exactly as specified in the{@link

	 * ReadableByteChannel} interface.</p>

	 */

   public abstract int read(ByteBuffer dst)throws IOException;

上面这个函数，就是与FileInputStream中read不同的地方了，通过读取channel当前位置起得一段字节到缓冲对象中，并且增加channel通道中当前位置的值。

	public abstract long read(ByteBuffer[] dsts,int offset, intlength)
							throws IOException;

从当前位置读取数据到多个缓冲对象中.offset以及length没有说明，回来可以看其他子类的实现来判定。

	public abstract int write(ByteBuffer src)throws IOException;

从缓冲对象中读取数据到channel的当前位置，如果channel是追加模式的话，就会追加到末尾，同时文件也会随之增大。

	public abstract long position() throws IOException;

获取当前通道的文件的位置，如果channel关闭了，则会抛出异常。

	public abstract FileChannel position(longnewPosition) throws IOException;

重新设置文件的位置。但是不会改变文件大小。
如果将位置设置在文件结束符之后，然后试图从文件通道中读取数据，读方法将返回-1 —— 文件结束标志。
如果将位置设置在文件结束符之后，然后向通道中写数据，文件将撑大到当前位置并写入数据。这可能导致“文件空洞”，磁盘上物理文件中写入的数据间有空隙。

	public abstract long size() throws IOException;

返回文件大小

	public abstract FileChannel truncate(longsize) throws IOException;

缩短channel的文件到指定的大小，如果比源文件大小大，则文件大小不变，如果比原文件大小小，则缩短文件到指定大小.可以使用FileChannel.truncate()方法截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除.

	public abstractlong transferTo(longposition,longcount,
                   WritableByteChanneltarget)
			throws IOException;

把这个channel流中的指定位置开始的指定长度，转移到另外一个writable通道中。

	public abstract long transferFrom(ReadableByteChannelsrc,
									long position, longcount)
							throws IOException;

把这个channel通道中的数据转到另外一个readable通道中。

	public abstract int read(ByteBuffer dst,long position) throws IOException;

把从指定位置开始的字节读入到字节缓冲对象中

	public abstract int write(ByteBuffer src,long position) throws IOException;

从指定位置开始写入ByteBuffer对象中.

FileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到FileChannel里的数据一定会即时写到磁盘上。要保证这一点，需要调用force()方法。

force()方法有一个boolean类型的参数，指明是否同时将文件元数据（权限信息等）写到磁盘上。

下面的例子同时将文件数据和元数据强制写到磁盘上：

	channel.force(true);

这个类后面还有一些其他方法不过，感觉太抽象了就不往后写了.要理解FileChannel的精髓，还要通过代码示例去验证这个方法的功能才能记得更深刻一些,到这里先总结一下这个类中用到的一些其他类，也是自己还没有看到的类：

	AbstractInterruptibleChannel、SeekableByteChannel、GatheringByteChannel、ScatteringByteChannel、

		Path、OpenOption、FileAttribute、ByteBuffer、WritableByteChannel、ReadableByteChannel。

要理解彻底关于FileChannel的用法，还要了解一下ByteBuffer这个对象模型才能搞定啊

	File file = new File("/users/terrylmay/desktop/in.txt");  
    File outFile = new File("/users/terrylmay/desktop/out.txt");  
    FileInputStream fis = new FileInputStream(file);  
    FileOutputStream fos = new FileOutputStream(outFile);  
    //通过文件流拿到通道  
    FileChannel channelIn = fis.getChannel();  
    FileChannel channelOut = fos.getChannel();  
    //为Buffer分配50个字节的空间  
    ByteBuffer buffer = ByteBuffer.allocate(50);  
    while(channelIn.read(buffer) != -1) {  
    //获取channel中的位置，这个位置应该是0  
    System.out.println("通道的当前位置:"+channelIn.position());  
    // public abstract int read(ByteBuffer dst) throws IOException;  

    //从channel的position读取一系列字节到缓冲对象中.  
    channelIn.read(buffer);  
    //这边打印channel通道的当前位置(打印值为50)  
    System.out.println("通道的当前位置:"+channelIn.position());  
    //同时可以看一下这时候channel.size()返回值是什么.(文件大小)  
    System.out.println("通道的size返回值:"+channelIn.size());  
    System.out.println("buffer中的数据:"+buffer.toString());  
    //更改buffer的模式--读模式.这时position变为limit，channel从buffer的0开始读到limit.  
    buffer.flip();  
    System.out.println("buffer中的数据:"+buffer.toString());  
    //这样会清空buffer中的数据  
    channelOut.write(buffer);  
    //positon为0表示ByteBuffer中已经没有数据了  
    System.out.println("buffer中的数据:"+buffer.toString());  
    buffer.clear();  

 本实例通过使用文件输入输出流获取的通道，从通道读数据到buffer中，并且从buffer中读取数据到输出channel中.

 在读取的时候必须调用buffer.flip()方法才能转换为读模式，否则可能读取数据出错的情况。

 具体原因是因为比如一个文件中有30个字节，设置buffer的空间为50byte,那么在从输入流中读取到buffer中的时候，会设置buffer中的position=30，limit=capacity=50，数据只存在与buffer中的0-30这部分.如果不调用flip()方法，则不会使limit = 30，position = 0，导致读取数据的时候还是从position开始读，到limit结束.就读不到正确的数据了.输出文件中就什么都不会复制过去.记得使用完之后要一次关闭文件流。
