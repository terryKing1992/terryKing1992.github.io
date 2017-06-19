---
layout: post
title: Java中Closeable和InputStream接口源码学习
date: 2016-02-21 09:50:05
tags: [Closeable, InputStream]
---

接口Closeable继承自AutoCloseable，同时重写了关于 close关闭流的方法。Closeable 是一个能够被关闭的数据源，close方法被调用区释放对象持有的资源例如打开的文件.
关闭stream流和与之相关的资源，如果流已经关闭了，那么再次调用这个方法是没有影响的。与AutoCloseable中的close方法一样，如果关闭操作失败，那么就要格外注意了。

<!-- more -->

当IO异常发生时该方法会抛出IOExce异常
	import java.io.IOException;  

	/**
	 * A {@code Closeable} is a source or destination of data that can be closed.
	 * The close method is invoked to release resources that the object is
	 * holding (such as open files).
	 *
	 * @since 1.5
	 */  
	public interface Closeable extends AutoCloseable {  

	    /**
	     * Closes this stream and releases any system resources associated
	     * with it. If the stream is already closed then invoking this
	     * method has no effect.
	     *
	     * <p> As noted in {@link AutoCloseable#close()}, cases where the
	     * close may fail require careful attention. It is strongly advised
	     * to relinquish the underlying resources and to internally
	     * <em>mark</em> the {@code Closeable} as closed, prior to throwing
	     * the {@code IOException}.
	     *
	     * @throws IOException if an I/O error occurs
	     */  
	    public void close() throws IOException;  
	}

上面主要是关于Closeable的介绍，而InputStream则是Closeable的一个实现，而InputStream是一个抽象类，所谓抽象类就是该类不能被实例化，只能实例化它的非抽象子类，我看的是jdk1.8的实现。

	public abstract class InputStream implements Closeable
	//MAX_SKIP_BUFFER_SIZE is used to determine the maximum buffer size to  
	use when skipping.  
	private static final int MAX_SKIP_BUFFER_SIZE = 2048;  

类中定义了一个static final类型的变量，用来指定InputStream流一次能够跳过的字节的最大值(忽略字节)，后面的一个方法skip会用到这个常量

	public abstract int read() throws IOException;

这个read方法待实现，主要是从流中读取一个字节作为输出，Reads the next byte of data from the input stream，因为0-255能够表示所有的字节，所以如果read方法返回-1，则说明这个流已经到了末尾。则不再读取即可.在while中直接break就OK了。

	public int read(byte b[]) throws IOException {  
	        return read(b, 0, b.length);  
	}

read(byte b[])方法读取流中的b.length个字节到字节数组b[]中。返回值表示读取到字节数组中的字节数。下面我们看一下关于read(b, 0, b.length)方法怎么实现的。

	public int read(byte b[], int off, int len) throws IOException {  
	    if (b == null) {  
	        throw new NullPointerException();  
	    } else if (off < 0 || len < 0 || len > b.length - off) {  
	        throw new IndexOutOfBoundsException();  
	    } else if (len == 0) {  
	        return 0;  
	    }  
	    int c = read();  
	    if (c == -1) {  
	        return -1;  
	    }  
	    b[off] = (byte)c;  

	    int i = 1;  
	    try {  
	        for (; i < len ; i++) {  
	            c = read();  
	            if (c == -1) {  
	                break;  
	            }  
	            b[off + i] = (byte)c;  
	        }  
	    } catch (IOException ee) {  
	    }  
	    return i;  
	}

(off表示开始位置，len表示读取流的最长字节)如果b字节数组为null的话，则会抛出空指针异常，如果开始位置off 小于0或者len小于0或者len > b.length - off(表示从开头off + 要读取的字节长度 len) 大于字节数组的length，则抛出数组越界异常。

如果len == 0，那么则直接返回0，表示读入到字节数组中的字节数为0；向后读取一个字节，如果返回为-1，则表示已经到达流的末尾，返回-1即可。b[off] = (byte)c;从这句代码就可以知道read(b, off, len)是把文件流读入到b字节数组中，读入数组中的位置从off索引开始，并且最长能够读取的就是填到b的末尾了，但是由于是从字节的off索引开始填充的，所以最长只能够读取b.length - off个字节。后面一个for循环，一个字节一个字节的读入到字节数组中off+i索引的位置。如果读到-1，那么跳出循环，返回读入到数组中的字节个数。

下面这个方法就是用到了最开始定义的最长能够跳过的字节数

	public long skip(long n) throws IOException {  

	    long remaining = n;  
	    int nr;  
	    if (n <= 0) {  
	        return 0;  
	    }  
	    int size = (int)Math.min(MAX_SKIP_BUFFER_SIZE, remaining);  
	    byte[] skipBuffer = new byte[size];  
	    while (remaining > 0) {  
	        nr = read(skipBuffer, 0, (int)Math.min(size, remaining));  
	        if (nr < 0) {  
	            break;  
	        }  
	        remaining -= nr;  
	    }  
	    return n - remaining;  
	}

跳过Math.min(MAX_SKIP_BUFFER_SIZE, remaining),这么长的字节，所谓的跳过，就是把这么多字节读入到另外一个不处理的字节数组中，然后函数结束的时候会自动释放，这样就跳过了一些流中的字节。比如要跳过3000字节，那么在调用Math.min方法的时候就会设置size=2048，然后调用read方法把2048个字节读入到skipBuffer字节数组中，假设文件流中只剩下了1000个字节，那么根据上面的方法介绍，则第一次进入while循环时，nr=1000，remaining=3000-1000=2000,因为n=3000，所以该函数返回3000-2000=1000，所以该函数返回的数据是跳过的字节数。

	public int available() throws IOException {  
	        return 0;  
	}

代码中是这么解释返回值的：返回一个可能被读取的流中的字节数或者能够跳过的流中字节数，该方法要其子类来实现，其实就是返回流中的字节数.

	public void close() throws IOException {}

这个方法就不用多说了，关闭文件流操作，不过具体实现要待子类中实现了。

	public synchronized void mark(intreadlimit) {}

同步操作，mark就像书签一样，在这个BufferedReader对应的buffer里作个标记，以后再调用reset时就可以再回到这个mark过的地方。mark方法有个参数，通过这个整型参数，你告诉系统，希望在读出这么多个字符之前，这个mark保持有效。读过这么多字符之后，系统可以使mark不再有效，而你不能觉得奇怪或怪罪它。这跟buffer有关，如果你需要很长的距离，那么系统就必须分配很大的buffer来保持你的mark。

	//reader       is       a       BufferedReader     
	reader.mark(50);//要求在50个字符之内，这个mark应该保持有效，系统会保证buffer至少可以存储50个字符     
	int       a       =       reader.read();//读了一个字符     
	int       b       =       reader.read();//又读了一个字符  

//做了某些处理，发现需要再读一次     

	reader.reset();     
	reader.read();//读到的字符和a相同     
	reader.read();//读到的字符和b相同

对应的使用reset则使得文件流指针又指向了原来mark的位置，读取相同数据

	public synchronized void reset() throws IOException {  
	    throw new IOException("mark/reset not supported");  
	}

该方法待子类实现。这里只抛出了不支持mark或者reset方法。

	public boolean markSupported() {  
	    return false;  
	}

测试这个文件流是否支持mark以及reset方法。

下面是测试代码：

	// TODO Auto-generated method stub
	//文件中的内容为：helloworld
	File file = new File(“/users/terrylmay/desktop/test.txt”);
	FileInputStream fis = new FileInputStream(file);
	BufferedInputStream br = new BufferedInputStream(fis);
	//br.skip(2);跳过两个字节，那么流中就只剩下了lloworld
	br.skip(2);
	//设置在读取的5个字节内标识有效，并且标记当前位置
	br.mark(5);
	//读取两个字节a = ‘l’ b=’l’
	int a = br.read();
	int b = br.read();
	//打印
	System.out.println(“”+(char)a+(char)b);
	//使得复位
	br.reset();
	//再次读取流中的字节
	a = br.read();
	b = br.read();
	//打印
	System.out.println(“”+(char)a+(char)b);
	//返回流中的剩余字节
	int length = br.available();
	System.out.println(length);

打印结果为:

	ll
	ll
	6
