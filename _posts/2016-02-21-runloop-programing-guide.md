---
layout: post
title: Runloop官方解释
date: 2016-04-20 21:46:52
tags: [Runloop, Thread Programming]
---

#前言

从2014年8月到公司报到，然后跟着导师维护Android SDK，再到后面跟着另外一名大牛Lance 学习IOS 的开发，
并且跟着项目走并且有人指导成长相对来说还是比较快的。但是对于接受新知识的能力还不足，具体表现在英语非常的不好；
而当时学习IOS的时候Lance和其他同事也是强烈建议读官方文档的，不过我却认为对于初学者 如果不是想紧随时代的步伐
(当然这也不会是一个好的程序员)，首先需要的是入门，而不是啃硬骨头。到现在有些知识理解了，并且经常实践了才会知道后面的原理。
当大概知道了某一些原理了之后再去读相关的官方文档，这样的好处是：第一：能够快速入门，不至于在没有入门的时候就丧失自信；
第二：对于自己理解有偏差的地方可以纠正；第三：对于自己的英文能力也是一种提高。而后面也可以跟着技术的步伐向前走了。

<!-- more -->

#Runloop是什么

Run loops are part of the fundamental infrastructure associated with threads.
A run loop is an event processing loop that you use to schedule work
and coordinate the receipt of incoming events. The purpose of a run loop is to
keep your thread busy when there is work to do and put your thread to sleep when there is none.

Runloop是与多线程相关的、基础框架中非常重要的一部分。
Runloop是你用来调度、协调当前的到来事件的一个循环。Runloop的目的就是当有任务到来的时候保持当前线程处于繁忙状态，
当没有任务需要处理的时候让当前线程处于休眠状态。


Run loop management is not entirely automatic.
You must still design your thread’s code to start the run loop at appropriate times
and respond to incoming events. Both Cocoa and Core Foundation provide run loop objects
to help you configure and manage your thread’s run loop.
Your application does not need to create these objects explicitly;
each thread, including the application’s main thread, has an associated run loop object.
Only secondary threads need to run their run loop explicitly, however.
The app frameworks automatically set up and run the run loop on the main thread as part
	of the application startup process.

Runloop 管理不是完全自动的。你还需要设计线程相关的代码 并且 在合适的时机开启当前线程的Runloop 去响应即将到来的事件。
不过还好 cocoa 和 core foundation框架都提供了一系列与runloop相关的对象让我们能够配置和管理线程的runloop。
我们的应用程序不需要显试的创建这些对象。每一个线程(包括主线程)都关联了一个runloop对象，
只有非主线程才需要显示的开启runloop处理不同数据源的事件。不过主线程的Runloop与非主线程不同，
应用程序在启动的时候就会自动的创建Runloop 并且跟着 应用的启动进程一起 启动并且运行在主线程。

The following sections provide more information about run loops and
how you configure them for your application.
For additional information about run loop objects, see NSRunLoop Class Reference and CFRunLoop Reference.

后面的章节将会告诉你关于run loop的更多信息 以及 如何在你的应用中配置runloop。
对于其他runloop 相关对象，请参考NSRunLoop 以及 CFRunLoop。

#Anatomy of a Run Loop(RunLoop分解)

A run loop is very much like its name sounds.
It is a loop your thread enters and uses to run event handlers in response to incoming events.
Your code provides the control statements used to implement the actual loop portion of the run loop—in other words,
your code provides the while or for loop that drives the run loop.
Within your loop, you use a run loop object to "run” the event-processing code
that receives events and calls the installed handlers.

RunLoop 的功能 与 它的名字十分相符。Runloop 用来响应 即将到来的事件，并且运行事件处理逻辑 响应相关事件。
你的代码提供了真正实现runloop的控制语句---换句话说，你的代码需要提供while循环或者for循环来驱动runLoop才可以让RunLoop处理事件。
在Runloop中，你需要使用一个runloop对象来"运行" 能够接收事件并且调用处理逻辑的 事件处理的代码。
RunLoop接收的事件来自于两个地方；第一:输入源 传递异步事件，一般来说 是来自其他线程 或者 其他应用的消息
第二:定时器源 传递同步事件 当timer调度开始的时候。这两种输入源都是用系统已经指定号的处理对象 来处理这些事件。

Figure 3-1 shows the conceptual structure of a run loop and a variety of sources.
The input sources deliver asynchronous events to the corresponding handlers
and cause the runUntilDate: method (called on the thread’s associated NSRunLoop object) to exit.
Timer sources deliver events to their handler routines but do not cause the run loop to exit.

如下图显示了Runloop概念结构图 和 多种多样的数据源。输入源传输异步事件到相应的处理模块
并且 使runUntilDate方法退出。定时器源 传递事件 到他们对应的处理模块，但是不会造成当前runloop退出。

In addition to handling sources of input, run loops also generate notifications
 about the run loop’s behavior. Registered run-loop observers can receive these notifications
 and use them to do additional processing on the thread.
 You use Core Foundation to install run-loop observers on your threads.

除了处理输入类型的事件之外，Runloop还会生成关于 Runloop相关行为的通知。
已经注册的Runloop的观察者 能够接收相应的 通知 并且在当前线程中处理额外的事情。
你可以使用Core Foundation中的方法 来 在你的线程中 注册相关的观察者 来处理不同的事件。

The following sections provide more information about the components of a run loop
and the modes in which they operate. They also describe the notifications
that are generated at different times during the handling of events

下面的段落 将提供 关于Runloop 组成的更多信息 以及 Runloop组成部分在哪种模式下运行。
下面还会描述 在处理不同的事件时产生的各种通知。

#Run Loop Modes (Runloop 运行模式)

A run loop mode is a collection of input sources and timers to be monitored
and a collection of run loop observers to be notified. Each time you run your run loop,
you specify (either explicitly or implicitly) a particular “mode” in which to run.
During that pass of the run loop, only sources associated with that mode are monitored and
allowed to deliver their events.
(Similarly, only observers associated with that mode are notified of the run loop’s progress.)
Sources associated with other modes hold on to any new events
until subsequent passes through the loop in the appropriate mode.

Runloop 模式 是一个集合，这个集合包含了 需要监控的输入源以及需要通知的观察者。
每次当你运行你的RunLoop，你都需要显式或者隐式的指定 当前线程所运行的模式。经过一次RunLoop之后，
只有与该 模式相关的 事件源才能被监听 以及 传递他们的事件。同样的，只有在当前模式下面的观察者才会被通知。
其他模式下的事件源，需要Hold住，并且当当前的Runloop 运行在 与挂起的事件源对应模式下面的时候，才会被处理。

In your code, you identify modes by name.
Both Cocoa and Core Foundation define a default mode and several commonly used modes,
along with strings for specifying those modes in your code.
You can define custom modes by simply specifying a custom string for the mode name.
Although the names you assign to custom modes are arbitrary, the contents of those modes are not.
You must be sure to add one or more input sources, timers,
or run-loop observers to any modes you create for them to be useful

在你的代码中，你需要根据名称指定线程RunLoop运行的模式。Cocoa 以及 CoreFoundation框架定义了一个默认的模式 以及一些常用的模式，
不过也需要在代码中通过字符串来指定这些模式。当然，你也可以自己设计特定的模式 通过指定自定义的字符串 当做 Mode的名称。
尽管你指定的的模式的名字是任意的，但是模式的内容却不是任意的。你必须为你自定义的模式 添加输入源 或者定时器 或者RunLoop 观察者 来确保Runloop可用。

You use modes to filter out events from unwanted sources during a particular pass through your run loop.
Most of the time, you will want to run your run loop in the system-defined “default” mode.
A modal panel, however, might run in the “modal” mode. While in this mode,
only sources relevant to the modal panel would deliver events to the thread. For secondary threads,
you might use custom modes to prevent low-priority sources from delivering events during time-critical operations.

当在一次循环中，你可以通过这些模式来过滤不想要的事件源。大部分时间，你都可以将你的runloop运行在默认模式下面，不过，
特殊场景需要运行在特殊的模式下才可以。对于非主线程，你需要使用自定义模式 来避免低优先级的事件源也 在一次循环中被处理。

Note: Modes discriminate based on the source of the event, not the type of the event.
For example, you would not use modes to match only mouse-down events or only keyboard events.
You could use modes to listen to a different set of ports, suspend timers temporarily,
or otherwise change the sources and run loop observers currently being monitored.

需要提醒的是：模式 只区分了 基于 事件的源，而并不指定事件的类型。举个栗子：你不能使用Modes来匹配鼠标的点击事件或者键盘事件。
你只可以使用模式 去监听一个端口的集合、暂时挂起的定时器、或者其他改变源 以及当前 被监听的RunLoop观察者。


#Input Source (输入源)

Input sources deliver events asynchronously to your threads. The source of the event depends on the type of the input source, which is generally one of two categories. Port-based input sources monitor your application’s Mach ports. Custom input sources monitor custom sources of events. As far as your run loop is concerned, it should not matter whether an input source is port-based or custom. The system typically implements input sources of both types that you can use as is. The only difference between the two sources is how they are signaled. Port-based sources are signaled automatically by the kernel, and custom sources must be signaled manually from another thread。

输入源 异步传递事件到你的线程中。事件源 取决于于某种特定类型的输入源，输入源有两种类型。第一种是基于端口的输入源监听你的应用程序的端口。自定义输入源监听自定义事件类型。只要你的RunLoop被输入源关联，RunLoop不关心到底是输入源还是自定义的。系统默认实现了两种类型的输入源供你使用。两种输入源之间的不同点就是一个需要信号通知，一个不需要。基于端口的输入源系统会自动发出信号，而自定义的输入源必须手动的在另外一个线程中手动发起信号。

When you create an input source, you assign it to one or more modes of your run loop. Modes affect which input sources are monitored at any given moment. Most of the time, you run the run loop in the default mode, but you can specify custom modes too. If an input source is not in the currently monitored mode, any events it generates are held until the run loop runs in the correct mode.

当你创建一个输入源的时候，你会将一个或者多个运行模式 赋值给你的RunLoop。运行模式 决定了在 某个特定的时候什么样的输入源会被监听以及调用。大部分情况下，你的RunLoop会运行在默认模式下，但是你也可以指定你自定义的模式。如果你的输入源不在当前监控的模式下，所有不在当前RunLoop运行模式下的事件都将会挂起，直到 RunLoop运行在正确的模式下才会得到执行。

##Port-Based Sources(基于端口的源)

Cocoa and Core Foundation provide built-in support for creating port-based input sources using port-related objects and functions. For example, in Cocoa, you never have to create an input source directly at all. You simply create a port object and use the methods of NSPort to add that port to the run loop. The port object handles the creation and configuration of the needed input source for you.

Cocoa 和 Core Foundation 为创建基于 端口的输入源 提供了API，只要你使用与端口相关的对象以及方法。例如：在Cocoa中，你并不需要直接创建一个输入源。你只需要简单的创建一个端口对象NSPort 以及使用NSPort中的一些方法，并且把该端口正在到RunLoop中即可。该端口对象会自动创建以及配置输入源所需要的一切。

In Core Foundation, you must manually create both the port and its run loop source. In both cases, you use the functions associated with the port opaque type (CFMachPortRef, CFMessagePortRef, or CFSocketRef) to create the appropriate objects.

在CoreFoundation框架中，你必须要手动的创建端口以及 RunLoop的事件源。你可以使用CFMachPortRef,CFMessagePortRef 以及CFSocketRef来创建合适的对象。

For examples of how to set up and configure custom port-based sources, see Configuring a Port-Based Input Source.

大家可以看如何创建以及配置一个自定义的基于端口类型的输入源,可以查看配置基于端口的输入源这一章。

##Custom Input Sources

To create a custom input source, you must use the functions associated with the CFRunLoopSourceRef opaque type in Core Foundation. You configure a custom input source using several callback functions. Core Foundation calls these functions at different points to configure the source, handle any incoming events, and tear down the source when it is removed from the run loop.

为了创建一个自定义的输入源，你需要使用在CoreFoundation中定义的CFRunLoopSourceRef类型。你需要使用多个回调函数来定义数据源。CoreFoundation在不同的时候调用这些方法来配置输入源，处理即将到来的时间以及销毁输入源当它从RunLoop中移除的时候。

In addition to defining the behavior of the custom source when an event arrives, you must also define the event delivery mechanism. This part of the source runs on a separate thread and is responsible for providing the input source with its data and for signaling it when that data is ready for processing. The event delivery mechanism is up to you but need not be overly complex.

另外，如果要为事件源定义当事件到来时候的行为，你需要定义事件的传递机制。这种机制运行在一个独立的线程中，根据输入源提供的数据 以及 当数据已经就绪之后的信号通知 来响应该事件。事件传递机制是由你决定但是不适合过于复杂。


For an example of how to create a custom input source, see Defining a Custom Input Source. For reference information for custom input sources, see also CFRunLoopSource Reference.

如下面例子中如何创建一个输入源，请参考Defining a Custom Input Source。


#Cocoa Perform Selector Sources

In addition to port-based sources, Cocoa defines a custom input source that allows you to perform a selector on any thread. Like a port-based source, perform selector requests are serialized on the target thread, alleviating many of the synchronization problems that might occur with multiple methods being run on one thread. Unlike a port-based source, a perform selector source removes itself from the run loop after it performs its selector.

另外,除了基于端口的事件源之外，Cocoa还定义了一个自定义的输入源，允许你将一个选择子丢到任意的线程中执行。与端口源一样，Perform Selector请求是在目标线程中串行执行的，为了避免多个方法同时运行在一个线程造成的同步问题。与端口源不同的一点是：perform selector事件源 会在调用完成之后自己从RunLoop中移除。

When performing a selector on another thread, the target thread must have an active run loop. For threads you create, this means waiting until your code explicitly starts the run loop. Because the main thread starts its own run loop, however, you can begin issuing calls on that thread as soon as the application calls the applicationDidFinishLaunching: method of the application delegate. The run loop processes all queued perform selector calls each time through the loop, rather than processing one during each loop iteration.

当在另外一个线程执行一个选择子的时候，目标线程必须有一个活跃的RunLoop。对于我们自己创建的线程，这意味着我们需要手动的开启RunLoop。因为主线程会自动开启它自己的RunLoop,但是你可以把mainThread 作为你的执行对象 只要在app 之执行了applicationDidFinishLaunching这个方法。RunLoop执行所有被放到队列中的选择子在RunLoop的每一次执行的时候，而不是经过一次RunLoop才会执行一个选择子。

Table 3-2 lists the methods defined on NSObject that can be used to perform selectors on other threads. Because these methods are declared on NSObject, you can use them from any threads where you have access to Objective-C objects, including POSIX threads. These methods do not actually create a new thread to perform the selector

表3-2中列出了NSObject定义的用来在其他线程中执行选择子的方法。因为这些方法定义在NSObject中，所以你可以在任意的线程中使用它，只要你能够访问OC对象，包括POSIX 线程。这些方法不会为了执行selector而创建一个新的线程。

	//Performs the specified selector on the application’s main thread during that thread’s next run loop cycle. These methods give you the option of blocking the current thread until the selector is performed.

	//执行一个特定的selector在应用的主线程的下一个RunLoop循环中。这些方法会阻塞当前线程知道selector执行完成。

	performSelectorOnMainThread:withObject:waitUntilDone:
	performSelectorOnMainThread:withObject:waitUntilDone:modes:


	//Performs the specified selector on any thread for which you have an NSThread object. These methods give you the option of blocking the current thread until the selector is performed.

	//执行一个特定的selector在任意线程，该线程中一定有一个NSThread对象。这些方法也会阻塞当前线程，知道选择子执行完成。

	performSelector:onThread:withObject:waitUntilDone:
	performSelector:onThread:withObject:waitUntilDone:modes:

	//Performs the specified selector on the current thread during the next run loop cycle and after an optional delay period. Because it waits until the next run loop cycle to perform the selector, these methods provide an automatic mini delay from the currently executing code. Multiple queued selectors are performed one after another in the order they were queued.

	//经过一定时间的延迟之后会在当前线程的下一次RunLoop循环中执行一个特定的selector。因为它需要等待直到下一次RunLoop循环去执行这个selector，所以下面的两个方法提供了自动延迟机制。多个被放到队列中的选择器进行一个接一个的顺序执行。
	performSelector:withObject:afterDelay:
	performSelector:withObject:afterDelay:inModes:

	Lets you cancel a message sent to the current thread using the performSelector:withObject:afterDelay: or performSelector:withObject:afterDelay:inModes: method

	让我们能够取消调用上面的接口发送到当前线程的消息

	cancelPreviousPerformRequestsWithTarget:
	cancelPreviousPerformRequestsWithTarget:selector:object:
