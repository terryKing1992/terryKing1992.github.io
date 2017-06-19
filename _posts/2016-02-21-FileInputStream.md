---
layout: post
title: Java中FileInputStream源码学习
date: 2016-02-21 09:42:45
tags: [FileInputStream]
---

前面两篇文章看到了接口和抽象类.基本上没有什么实现，除了skip(long n) 以及read(),readBytes(b[]),read(b, off, len)其他的基本上什么都没说到，今天我终于看到了一些具体的实现.除了那些接口之外的一些实现.

可以说文件流的第一层封装应该是FileInputStream这个层面上的封装了。下面看一下这里面主要实现了哪些方法，以及如何实现的。

<!-- more -->

首先，类定义自然不用说了。public class FileInputStreamextends InputStream，继承自抽象类InputStream，同时继承了里面的read 、readBytes、skip等方法，不知道该类中是否重写了这些方法呢?带着一系列的疑问，来开始对FileInputStream的学习。

	/ File Descriptor - handle to the open file /
	private final FileDescriptor fd;

解释：文件描述符–可以说是文件句柄，用来打开文件。

	/**
	 * The path of the referenced file
	 * (null if the stream is created with a file descriptor)
	 */

	private final String path;

解释：引用文件的路径(该流引用的文件)，如果是通过文件描述符来创建的文件流的话，那么这个值为null。

	private FileChannelchannel =null;

解释：文件通道–这个类还不是很清楚是干什么的,根据字面意思就是文件管道–应该能够双向读写？

	private final Object closeLock = new Object();

解释：当流调用close方法的时候，可能要再方法内代码块上加锁，具体为什么加锁，以及怎么加的锁可以后面看代码

	private volatile boolean closed =false;

解释：标识流的关闭状态

	public FileInputStream(Stringname)throws FileNotFoundException {
	   this(name !=null ?new File(name) :null);
	}

通过文件名(相对系统的路径)，如果文件不存在或者文件不是一个普通文件而是一个目录或者由于某些原因文件不能被打开，那么就会抛出FileNotFoundException异常。然后调用本类的参数为File的构造方法

	public FileInputStream(Filefile)throws FileNotFoundException {
	    Stringname = (file !=null ?file.getPath() : null);
		//TODO 安全管理这个回头去System类中看一下
	   SecurityManagersecurity = System.getSecurityManager();
	   if (security !=null) {
	       security.checkRead(name);
	   }
	   if (name ==null) {
	       thrownew NullPointerException();
	   }
	   if (file.isInvalid()) {
	       thrownew FileNotFoundException("Invalid file path");
	   }
	   fd = new FileDescriptor();
	   fd.attach(this);
	   path = name;
	   open(name);
	}

如果文件为空则抛出FileNotFoundException(非法路径)，然后将文件描述符fd与该文件流绑定，并且文件路径赋值给path，然后打开文件流。点击open(name)这个方法，卧槽，竟然是本地方法。这尼玛是逼着我去看c的实现啊..

	public FileInputStream(FileDescriptorfdObj) {
	   SecurityManagersecurity = System.getSecurityManager();
	   if (fdObj ==null) {
	       thrownew NullPointerException();
	   }
	   if (security !=null) {
	       security.checkRead(fdObj);
	   }
	   fd = fdObj;
	   path = null;
	   /*
	    * FileDescriptor is being shared by streams.
	    * Register this stream with FileDescriptor tracker.
	    */
	   fd.attach(this);
	}

这个方法就解释了为什么参数是文件描述符来创建文件流，path为null，文件描述符被多个流共享，则把这个流放入到FileDescriptor的追踪项里面(应该就是一个链表中)。

	publicint read() throws IOException {
	   return read0();
	}

	private native int read0()throws IOException;

我晕，又是本地方法调用..

	private native int readBytes(byteb[], int off,int len) throws IOException;
	public int read(byte b[])throws IOException {
	       return readBytes(b, 0,b.length);
	}
	public int read(byteb[],int off,intlen) throws IOException {
	    return readBytes(b,off, len);
	}

读取文件操作.果然重写了父类中的read方法。

	publicnativelong skip(longn)throws IOException;

父类中有实现，点开实现，看到在SocketInputStream中也有相关实现，不过在SocketInputStream实现中只能一次忽略1024个字节，具体实现如下：
	 public long skip(long numbytes)throws IOException {
	   if (numbytes <= 0) {
	       return 0;
	    }
	   long n = numbytes;
	   int buflen = (int) Math.min(1024,n);
	   byte data[] = newbyte[buflen];

	   while (n > 0) {
	       intr = read(data, 0, (int) Math.min((long)buflen,n));
	       if (r < 0) {
	           break;
	       }
	       n -=r;
	    }
	   return numbytes - n;
	}

跟InputStream唯一一点的不同就是1024与2048的差别了。

	public void close() throws IOException {
	   synchronized (closeLock) {
	       if (closed) {
	           return;
	        }
	       closed =true;
	    }

	   if (channel !=null) {
	      channel.close();
	    }

	   fd.closeAll(new Closeable() {
	       publicvoid close() throws IOException {
	           close0();
	       }
	    });
	}

这里把与文件流相关的FileChannel以及文件描述符相关资源全部关闭，但是这个在关闭代码块中为什么要加上同步锁，这点就不是很懂了.欢迎指导~

	public FileChannel getChannel() {
	   synchronized (this) {
	       if (channel ==null) {
	           channel = FileChannelImpl.open(fd,path,true, false,this);
	        }
	       returnchannel;
	    }
	}

拿到文件流通道，一个文件流只能创建一个文件管道，这里使用同步方法还是比较好理解的.

 	protected void finalize() throws IOException {
	   if ((fd !=null) &&  (fd != FileDescriptor.in)) {
	       /* if fd is shared, the references in FileDescriptor
	         * will ensure that finalizer is only called when
	         * safe to do so. All references using the fd have
	         * become unreachable. We can call close()
	         */
	        close();
	    }
	}

当这个文件流被垃圾回收机制回收的时候，会调用这个finalize()方法，用来做一些资源的释放操作.

基本上用FileInputStream这个类也主要是读取文件中的字节数，使用read、read(b, off, len)方法就可以了.

还没有用到其中的其他方法，不过还有一些方法就是skip、getChannel方法同样能够用到.

如果对FileChannel更加了解了，感觉这个比自己只使用read方法更加高效一点.
