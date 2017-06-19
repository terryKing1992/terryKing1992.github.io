---
title: GCD API详解
date: 2016-02-21 11:11:21
tags: [GCD, 多线程]
---

# **前言:GCD API 详解**

GCD 在IOS 多线程编程中起着举足轻重的作用；GCD相对来说是更加抽象化的API，但是它比较靠近底层一些，都是使用C语言来实现的。下面我们一起走进GCD的世界，探索它里面的奥秘；

<!-- more -->

# **一、Dispatch Queue**

从名字(分发队列)我们就可以看出，这个是任务都会进入到这个队列中等待处理。开发者是需要把要执行的任务添加到该队列中即可。

	{% codeblock lang:objc %}
	dispatch_async(dispatch_get_main_queue(), ^{
	    NSLog(@"欢迎来到GCD的世界");
	});
	//我们可以把一个任务放到异步队列中去，当然也可以把任务放到同步队列中，比如下面的代码：
	//不过下面这种方式如果在主线程执行会造成死锁的情况,原理如下:
	//dispatch_sync只有等到block执行完成才会返回执行后面NSLog(@"End");,而block又是追加到mainQueue的末端的,末端的任务需要等到NSLog(@"End");任务执行完成之后才会执行,这样就造成了循环等待的情况,从而造成死锁
	dispatch_sync(dispatch_get_main_queue(), ^{
	    NSLog(@"欢迎来到GCD的世界");
	});
	NSLog(@"End");
	{% endcodeblock %}

从这两个函数名称就可以看出，第一个 是另外开了一个线程，将任务丢进了了主队列,第二个任务 是在当前线程 把任务丢到 主队列中

# **二、Dispatch_Queue_Serial 和 Dispatch_Queue_Concurrent**

对于串行队列与并发队列的区别，串行队列中的任务具有先后执行顺序，而并行队里中的任务能够并发执行。
在开始写代码之前，我们需要先了解另外一个API dispatch_queue_create,这个API接收两个参数，第一个表示该队里的标示，第二个表示该队列类型(并行还是串行)；

下面我们先看一下怎么创建并行以及串行队列：
	
	{% codeblock lang:objc %}
	//创建串行队列
	dispatch_queue_t serialQueue = dispatch_queue_create("com.huawie.serialQueue", DISPATCH_QUEUE_SERIAL);
	//创建并行队列
	dispatch_queue_t concurrentQueue = dispatch_queue_create("com.huawei.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
	了解了并行队列以及串行队列的创建方法之后，让我们一起看一下怎么将任务放到串行以及并行队列；
	//下面这段代码的作用是把两个任务放入到串行队列中，两个任务联合工作打印出比5小的正整数

	dispatch_queue_t serialQueue = dispatch_queue_create("com.example.gcd.serialQueue", DISPATCH_QUEUE_SERIAL);

	__weak typeof(self)weakSelf = self;

	__block int i = 1;
	void(^block)() = ^ {
	    while (i < 5) {
	        @synchronized(weakSelf) {
	            NSLog(@"thread 1: i = %d", i);
	            i++;
	        }
	    }
	};

	void(^block2)() = ^ {
	    while (i < 5) {
	        @synchronized(weakSelf) {
	            NSLog(@"thread 2: i = %d", i);
	            i++;
	        }
	    }
	};

	dispatch_async(serialQueue, block);
	dispatch_async(serialQueue, block2);
	{% endcodeblock %}


	//打印结果如下：

	2015-12-19 16:03:14.028 thread 1: i = 1
	2015-12-19 16:03:14.028 thread 1: i = 2
	2015-12-19 16:03:14.029 thread 1: i = 3
	2015-12-19 16:03:14.029 thread 1: i = 4

明显可以看出：在程序执行完第一个block之后，i的值已经为5了。所以，第二个任务并没有进入while循环就退出了。说明两个任务是顺序执行的。
下面我们直接将串行队列修改为并行队列，看一下效果：
	
	{% codeblock lang:objc %}
	//下面这段代码的作用是把两个任务放入到并行队列中，两个任务联合工作打印出比5小的正整数
	//创建并行队列
	dispatch_queue_t concurrentQueue = dispatch_queue_create("com.huawei.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);

	__weak typeof(self)weakSelf = self;

	__block int i = 1;
	void(^block)() = ^ {
	    while (i < 5) {

	        //1 标识
	        @synchronized(weakSelf) {
	            NSLog(@"thread 1: i = %d", i);
	            i++;
	        }
	    }
	};

	void(^block2)() = ^ {
	    while (i < 5) {
	        @synchronized(weakSelf) {
	            NSLog(@"thread 2: i = %d", i);
	            i++;
	        }
	    }
	};

	dispatch_async(concurrentQueue, block);
	dispatch_async(concurrentQueue, block2);
	{% endcodeblock %}

	这个打印更奇怪了，先看打印吧：
	2015-12-19 16:15:39.535  thread 1: i = 1
	2015-12-19 16:15:39.536  thread 2: i = 2
	2015-12-19 16:15:39.536  thread 1: i = 3
	2015-12-19 16:15:39.537  thread 2: i = 4
	2015-12-19 16:15:39.540  thread 1: i = 5

竟然能打出来i = 5，真是吓了我一跳，不过仔细分析之后也能发现问题:

当i = 4的时候，线程1 走到了1标识处，然后cpu被线程2抢占，线程2执行完成之后，i = 5，这时候，线程1走到了打印的地方，就变成了5，并且打印出来了。


所以，对于上面的程序，为了避免出现多线程访问程序出错的问题，修改程序代码如下：

	{% codeblock lang:objc %}
	//创建并行队列
	dispatch_queue_t concurrentQueue = dispatch_queue_create("com.huawei.concurrentQueue", DISPATCH_QUEUE_CONCURRENT);

	__weak typeof(self)weakSelf = self;


	__block int i = 1;
	void(^block)() = ^ {
	    while (i < 5) {
	        @synchronized(weakSelf) {
	            if (i < 5) {
	                NSLog(@"thread 1: i = %d", i++);
	            }else {
	                break;
	            }
	        }
	    }
	};

	void(^block2)() = ^ {
	    while (true) {
	        @synchronized(weakSelf) {
	            if (i < 5) {
	                NSLog(@"thread 2: i = %d", i++);
	            }else {
	                break;
	            }
	        }
	    }
	};

	dispatch_async(concurrentQueue, block);
	dispatch_async(concurrentQueue, block2);
	{% endcodeblock %}

这样程序就运行正常了。

# **三、Main Dispatch Queue 和 Global Dispatch Queue**

这两个队列是由系统提供的。我们就只管用就可以了，这两者之间的区别在于主线程队列 是一个 串行队列，而全局队列 是一个并行队列。
一般操作UI 都会派发到主线程队列中去执行。下面我们看一下这两种队列该如何获取和使用：


	{% codeblock lang:objc %}
	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
	    NSLog(@"先在全局并行队列打印");
	    dispatch_async(dispatch_get_main_queue(), ^{
	       NSLog(@"再到主线程队列打印");
	    });
	    dispatch_async(dispatch_get_main_queue(), ^{
	        NSLog(@"就是这么任性");
	    });
	});
	{% endcodeblock %}
	

	还是先看输出，在解释代码：

	2015-12-19 16:33:38.822  先在全局并行队列打印
	2015-12-19 16:33:38.822  再到主线程队列打印
	2015-12-19 16:33:38.822  就是这么任性

先看主线程队列的创建，或者也不能叫创建，因为它本身就已经存在了，应该叫获取。

只需要调用一句话即可：

	{% codeblock lang:objc %}
	dispatch_get_main_queue()
	{% endcodeblock %}
	

而全局队列就比较繁琐一点,当然如果你想简单一点也可以。global 中 有线程的优先级，包括高优先级(DISPATCH_QUEUE_PRIORITY_HIGH), 默认优先级(DISPATCH_QUEUE_PRIORITY_DEFAULT),低优先级(DISPATCH_QUEUE_PRIORITY_LOW),后台优先级(DISPATCH_QUEUE_PRIORITY_BACKGROUND)

这四个变量只是用来指定线程执行的优先级高低，当然，我们也可以只用默认优先级的就可以了。获取全局线程队列：

	{% codeblock lang:objc %}
	dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);
	{% endcodeblock %}


如果大家有印象的话，前面还说到了另外一个GCD 的API dispatch_queue_create,创建队列。那么这个API 创建出来的队列与 已经存在的这两个队列 又有什么区别呢?自己创建的队列 是不需要指定优先级的，即默认优先级，如果要指定具体的优先级，那么还需要另外一个API 来辅助完成。请看下一节；

# **四、dispatch_set_target_queue**

- 这个API 能够指定自己创建的queue的优先级，具体怎么使用，请看代码：
	//需要改变优先级的队列
	dispatch_queue_t serialQueue = dispatch_queue_create(“com.huawie.serialQueue”, DISPATCH_QUEUE_CONCURRENT);
	//具有优先级的队列
	dispatch_queue_t globalQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0);

	//将第一个参数对应的队列的优先级 设置与 globalQueue的优先级相同
	dispatch_set_target_queue(serialQueue, globalQueue);

- 这个API 同样能够 将并行执行的 多个serialQueue 
放到一个串行队列中去，让这几个本来应该并行执行的串行队列能够串行执行。

这句话可能有点绕啊。可能代码更加清晰一些；

## **第一种场景：多个串行队列 并行执行任务**

	{% codeblock lang:objc %}
	dispatch_queue_t serialQueue1 = dispatch_queue_create("1", DISPATCH_QUEUE_SERIAL);
	dispatch_queue_t serialQueue2 = dispatch_queue_create("2", DISPATCH_QUEUE_SERIAL);
	dispatch_queue_t serialQueue3 = dispatch_queue_create("3", DISPATCH_QUEUE_SERIAL);
	dispatch_queue_t serialQueue4 = dispatch_queue_create("4", DISPATCH_QUEUE_SERIAL);

	dispatch_async(serialQueue1, ^{
	    [NSThread sleepForTimeInterval:1];
	    NSLog(@"1");
	});

	dispatch_async(serialQueue2, ^{
	    NSLog(@"2");
	});

	dispatch_async(serialQueue3, ^{
	    NSLog(@"3");
	});

	dispatch_async(serialQueue4, ^{
	    NSLog(@"4");
	});
	{% endcodeblock %}

因为考虑到程序执行过快，导致感觉正确的输出，我让第一个队列休眠1s钟。我们可以看到输出为：

	2015-12-19 17:02:21.802  2
	2015-12-19 17:02:21.802  3
	2015-12-19 17:02:21.802  4
	2015-12-19 17:02:22.803  1

而怎么样才能让改程序输出1、2、3、4呢？别担心，这个API 能够帮我们搞定。

	{% codeblock lang:objc %}
	dispatch_queue_t serialQueue1 = dispatch_queue_create("1", DISPATCH_QUEUE_SERIAL);
	dispatch_queue_t serialQueue2 = dispatch_queue_create("2", DISPATCH_QUEUE_SERIAL);
	dispatch_queue_t serialQueue3 = dispatch_queue_create("3", DISPATCH_QUEUE_SERIAL);
	dispatch_queue_t serialQueue4 = dispatch_queue_create("4", DISPATCH_QUEUE_SERIAL);

	dispatch_queue_t serialMain = dispatch_queue_create("main", DISPATCH_QUEUE_SERIAL);

	//将串行队列放入到另外一个大的串行队列中去。
	dispatch_set_target_queue(serialQueue1, serialMain);
	dispatch_set_target_queue(serialQueue2, serialMain);
	dispatch_set_target_queue(serialQueue3, serialMain);
	dispatch_set_target_queue(serialQueue4, serialMain);

	dispatch_async(serialQueue1, ^{
	    [NSThread sleepForTimeInterval:1];
	    NSLog(@"1");
	});

	dispatch_async(serialQueue2, ^{
	    NSLog(@"2");
	});

	dispatch_async(serialQueue3, ^{
	    NSLog(@"3");
	});

	dispatch_async(serialQueue4, ^{
	    NSLog(@"4");
	});
	{% endcodeblock %}

# **五、dispatch_sync**

这个方法跟dispatch_async 的主要区别，打个比喻吧：快递小哥给你打电话，让你去取快递，快递小哥打过电话之后就去处理其他事情了，并没有一直在那什么都做得等着你去领快递。这种就是async，否则就是sync。

用程序解释一下就是：

	{% codeblock lang:objc %}
	//4
	dispatch_async(serialQueue4, ^{
	    NSLog(@"4");
	});

	//5
	dispatch_sync(serialQueue4, ^{
	    NSLog(@"5");
	});

	//6
	NSLog(@"6");
	{% endcodeblock %}
	

async 在4结束之后马上就会执行5，不管4的block有没有加入到serialQueue4队列中去，而sync 必须等到5的block加入到serialQueue4中才会执行6.

# **六、Dispatch_After函数**

这个函数主要用在某个函数需要等待多少s之后执行。下面我们直接看一下代码怎么使用：

	{% codeblock lang:objc %}
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
	    NSLog(@"我停了1s才打印哦~");
	});
	{% endcodeblock %}
	

其实所有的想队列中最佳任务的操作都需要经过Runloop,假设系统的Runloop执行时间为1s，每隔1s执行一次Runloop，那么如果要把block添加到队列中去，加上延迟的时间最长需要3s的时间才能够加入到queue中。其实系统的Runloop也是挺有趣的，回头一定要研究一下。

# **七、Dispatch Group**

这种主要是在一组操作结束之后才能做其他事情函数。比如我们要进行网络数据的获取，分了3个线程去拉取数据，等到所有的线程都结束之后，再去更新UI操作。
下面我们看一下简单的代码实现：

	{% codeblock lang:objc %}
	dispatch_group_t group = dispatch_group_create();
	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

	dispatch_group_async(group, queue, ^{
	    NSLog(@"完成第一步");
	});

	dispatch_group_async(group, queue, ^{
	    NSLog(@"完成第二步");
	});

	dispatch_group_async(group, queue, ^{
	    NSLog(@"完成第三步");
	});

	dispatch_group_notify(group, queue, ^{
	    NSLog(@"全部操作已经完成,回家睡觉");
	});
	{% endcodeblock %}
	
	输出如下：
	DotaInstruction[2004:206186] 完成第一步
	2015-12-19 17:47:50.378 DotaInstruction[2004:206169] 完成第三步
	2015-12-19 17:47:50.378 DotaInstruction[2004:206187] 完成第二步
	2015-12-19 17:47:50.379 DotaInstruction[2004:206169] 全部操作已经完成,回家睡觉

从上面输出我们可以看到，组里面的操作完成的时间是不确定的。

但是有一点就是block中不能再写一个异步的操作了，不然这个group 是不知道 你的异步操作完成了没有的。举个栗子：

	{% codeblock lang:objc %}
	dispatch_group_t group = dispatch_group_create();
	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

	dispatch_group_async(group, queue, ^{
	    NSLog(@"完成第一步");
	});

	dispatch_group_async(group, queue, ^{
	    dispatch_async(dispatch_get_main_queue(), ^{
	        [NSThread sleepForTimeInterval:2];
	        NSLog(@"完成第二步");
	    });
	});

	dispatch_group_async(group, queue, ^{
	    NSLog(@"完成第三步");
	});

	dispatch_group_notify(group, queue, ^{
	    NSLog(@"全部操作已经完成,回家睡觉");
	});
	{% endcodeblock %}

	我们可以看到打印：

	DotaInstruction[2029:209120] 完成第一步
	2015-12-19 17:51:34.202 DotaInstruction[2029:209121] 完成第三步
	2015-12-19 17:51:34.204 DotaInstruction[2029:209121] 全部操作已经完成,回家睡觉
	2015-12-19 17:51:36.212 DotaInstruction[2029:209055] 完成第二步

这么写代码造成了group觉得在dispatch_async 之后就已经结束了，并不关心你异步线程里面的事情是否做完了。所以会有如上打印。

# **八、dispatch_once**

保证代码只执行一次，最常见的是用在单例上面。下面是单例的一种实现。


	{% codeblock lang:objc %}
	+ (instancetype)sharedInstance {
	    static Singleton *instance = nil;
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        instance = [[super allocWithZone:NULL] init];
	    });
	    return instance;
	}

	+ (instancetype)allocWithZone:(struct _NSZone *)zone {
	    return [self sharedInstance];
	}
	{% endcodeblock %}
	

到这里基本上常用的GCD的API都已经完了，还有几个跟高效读取文件系统数据相关，比如dispatch_barrier_async、dispatch_io,这个后面要再研究一下。
