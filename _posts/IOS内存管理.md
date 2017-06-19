---
title: IOS内存管理
date: 2016-02-21 11:04:49
tags: [IOS, 内存管理, 属性修饰符]
---

# **前言**

内存管理是计算机编程最为基本的领域之一。在很多脚本语言中，您不必担心内存是如何管理的，这并不能使得内存管理的重要性有一点点降低。对实际编程来说，理解您的内存管理器的能力与局限性至关重要。在大部分系统语言中，比如 C 和C++，您必须进行内存管理。本文将介绍手工的、自动的内存管理实践的基本概念。

<!-- more -->

# **一、属性修饰符简介**

说到内存管理，不可避免的要提到属性修饰符。在本章中我们只关心跟内存管理相关的修饰符：

- assign: 主要是用来修饰基本数据类型
- copy:copy 可以用来修饰NSString 或者 对象类型(如果修饰对象类型,那么被修饰的对象一定要实现NSCopying协议)
- weak: 为了防止循环引用,引入的概念.被weak修饰 引用计数不变
- retain: MRC中引入的概念,在ARC中也可以使用.但最好用strong
- strong: 对象的修饰符

# **二、 assign修饰符介绍**

assign 用来修饰基本数据类型，比如NSInteger、BOOL、int等数据类型.

- **场景一: 直接修饰成员属性,其修饰成员的生命周期随对象的释放而释放**

我们创建一个Person类,里面包含了两个属性:isCoder表示是否是程序员，age表示年龄

	{% codeblock Person头文件 lang:objc %}
		@interface Person : NSObject

		@property (nonatomic, assign) BOOL isCoder;
		@property (nonatomic, assign) NSInteger age;

		@end
	{% endcodeblock %}

	{% codeblock Person实现类 lang:objc %}
		#import "Person.h"

		@implementation Person

		@end
	{% endcodeblock %}

我们根据具体的代码示例开始assign之旅

	{% codeblock assign示例代码 lang:objc %}
		NSInteger age = 24;
		BOOL isCoder = YES;
		person.isCoder = isCoder;
		person.age = age;
	{% endcodeblock %}
	

这4行代码执行之后,person中的属性值发生了改变.

但是 因为age 与 isCoder并不是对象类型,它们也没有retainCount的属性.所以并不存在引用计数增加的情况.


- **场景二：assign修饰**对象类型**会发生什么情况**

我们顺着上面的代码往下走,另外再定义World类.

	{% codeblock World类头文件 lang:objc %}
		#import <Foundation/Foundation.h>

		#import "Person.h"

		@interface World : NSObject

		@property (nonatomic, assign) Person *person;

		@end
	{% endcodeblock %}

	{% codeblock World实现类 lang:objc %}
		#import "World.h"

		@implementation World

		@end
	{% endcodeblock %}

下面开始看assign修饰对象代码如何来写:

	{% codeblock Assign修饰对象代码 lang:objc %}
		Person *person = [[Person alloc] init];
	    person.isCoder = YES;
	    person.age = 24;
	    
	    World *world = [[World alloc] init];
	    world.person = person;
	    
	    NSLog(@"此时World类中person属性的值为:%@", world.person);
	{% endcodeblock %}

这时候我们可以看到,world.person对象是有值的,那么因为上面声明的person对象还没有释放.

这时候我们使用lldb控制台打印一下world.person.retainCount可以看到一下输出:

	此时World类中person属性的值为:<Person: 0x7fa592e08a80>
	(lldb) po person.retainCount
	**1**

说明assign修饰的对象,引用计数是不会增加的.为了更好的验证assign修饰的对象,
在超出赋值运算符右边对象(比如:person对象)生命周期之后就会自动变为nil.我们把world对象改变成成员变量.在AppDelegate承载:

	{% codeblock AppDelegate类代码 lang:objc %}
		#import "Person.h"
		#import "World.h"

		@interface AppDelegate ()
		@property (nonatomic, strong) World *world;
		@end

		@implementation AppDelegate


		- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
		    // Override point for customization after application launch.
		    
		    Person *person = [[Person alloc] init];
		    person.isCoder = YES;
		    person.age = 24;
		    
		    self.world = [[World alloc] init];
		    self.world.person = person;
		    
		    //**当在该didFinishLaunchingWithOptions 还能够访问到person对象,因为person对象还没有出作用域**
		    NSLog(@"此时World类中person属性的值为:%@", self.world.person);
		    
		    return YES;
		}
		- (void)applicationDidBecomeActive:(UIApplication *)application {
		    // Restart any tasks that were paused (or not yet started) while the application was inactive. If the application was previously in the background, optionally refresh the user interface.
		    //**当到该方法的时候,因为已经出了person对象的作用域,这时候在看world.person对象是什么状态**
		    NSLog(@"在%s 方法中self.world.person的值为:%@", __func__, self.world.person);
		}
		@end
	{% endcodeblock %}

当程序进入applicationDidBecomeActive方法的时候,因为person已经出了作用域.所以,引用计数已经变为0.

不过此时的self.world.person还指向了原来person指向的地址.这时候就会出现野指针的情况导致程序崩溃.

代码执行完毕之后，如果在World类的其他方法中调用self.person.age,看一下效果。这时候调用self.person.age 因为self.person 指向的内存地址被其他的对象占用了，造成无法预知的错误。有可能内存没有被其他对象使用，因为内存地址中已经没有任何Person的信息了，造成的错误提示为：

    Thread-1:Bad-Exe-Acces

这时候我们打印堆栈信息可以发现:
	
	thread #1: tid = 0xb196, 0x0000000105f2cc78 libobjc.A.dylib`realizeClass(objc_class*) + 41, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x7e8)
    frame #0: 0x0000000105f2cc78 libobjc.A.dylib`realizeClass(objc_class*) + 41
    frame #1: 0x0000000105f2cd59 libobjc.A.dylib`realizeClass(objc_class*) + 266
    frame #2: 0x0000000105f2fcdb libobjc.A.dylib`lookUpImpOrForward + 104
    frame #3: 0x0000000105f3e8bb libobjc.A.dylib`objc_msgSend + 187

objc调用了消息传递机制,最终还是没能找到响应方法的对象,最后Crash

**所以，为了避免不可预知的错误，在修饰对象的时候一定不要使用assign**

---

# **三、copy修饰符介绍**
 
## **场景一、copy修饰符通常用在NSString对象上面**

我们在上面Person类的基础上增加一个name的属性


	{% codeblock Person头文件 lang:objc %}
		#import <Foundation/Foundation.h>

		@interface Person : NSObject

		@property (nonatomic, assign) BOOL isCoder;
		@property (nonatomic, assign) NSInteger age;
		@property (nonatomic, copy) NSString *name;

		@end
	{% endcodeblock %}
	
	{% codeblock Copy示例代码 lang:objc %}
		 NSString *name = @"terry";
    
	    Person *person = [[Person alloc] init];
	    person.isCoder = YES;
	    person.age = 24;
	    person.name = name;
	    
	    NSLog(@"person.name == name 结果为:%d", person.name == name);
	{% endcodeblock %}

大家应该能够猜到结果了

	person.name == name 结果为:1

大家都知道 == 用来比较内存地址,这个结果表明 copy修饰 NSString的话,实际上与strong 修饰NSString 效果是一样的.

## **场景二、当然也可以用在自定义对象以及Foundation框架中的其他对象上面,下面我们通过代码的形式解读copy修饰其他对象**

上面也提到过,如果想使用copy修饰符来修饰自定义对象的话,自定义对象一定要实现NSCopying协议才可以.

不实现的话,当为world.person赋值的时候会crash

	-[Person copyWithZone:]: unrecognized selector sent to instance 0x7f93b8d18ad0
	2016-05-16 23:30:51.540 MicroTalk[7351:56161] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[Person copyWithZone:]: unrecognized selector sent to instance 0x7f93b8d18ad0'
	*** First throw call stack:
	(
		0   CoreFoundation                      0x0000000105b66e65 __exceptionPreprocess + 165
		1   libobjc.A.dylib                     0x00000001052dddeb objc_exception_throw + 48
		2   CoreFoundation                      0x0000000105b6f48d -[NSObject(NSObject) doesNotRecognizeSelector:] + 205
		3   CoreFoundation                      0x0000000105abc90a ___forwarding___ + 970
		4   CoreFoundation                      0x0000000105abc4b8 _CF_forwarding_prep_0 + 120
		5   libobjc.A.dylib                     0x00000001052ebb93 objc_setProperty_nonatomic_copy + 39
		6   MicroTalk                           0x0000000101ca18c7 -[World setPerson:] +

 
我们把World类中的assign修饰符改为copy
	
	{% codeblock World头文件代码 lang:objc %}
		 #import "Person.h"

		@interface World : NSObject

		@property (nonatomic, copy) Person *person;

		@end
	{% endcodeblock %}

	{% codeblock Person类代码 lang:objc %}
		@interface Person : NSObject<NSCopying>

		@property (nonatomic, assign) BOOL isCoder;
		@property (nonatomic, assign) NSInteger age;
		@property (nonatomic, copy) NSString *name;

		@end

		@implementation Person

		- (instancetype)copyWithZone:(NSZone *)zone {
		    return self;
		}
		@end
	{% endcodeblock %}

	{% codeblock copy示例代码 lang:objc %}
		NSString *name = @"terry";
    
	    Person *person = [[Person alloc] init];
	    person.isCoder = YES;
	    person.age = 24;
	    person.name = name;
	    
	    self.world = [[World alloc] init];
	    
	    self.world.person = person;
        NSLog(@"self.world.person == person 结果为:%d", self.world.person == person);
	{% endcodeblock %}

打印结果为:

	self.world.person == person 结果为:1

如果Person中的copyWithZone 是上面这种实现，这时候，你就会发现self.person的引用计数目前已经是2了。

因为Person的copy方法返回的还是自己。如果这种代码出现在MRC中的话，就会造成内存泄露。在MRC中，给self.person的正确赋值方式应嘎是下面的情况

	{% codeblock Copy示例代码 lang:objc %}
		 self.person = [[[Person alloc] init] autorelease];`
	{% endcodeblock %}

这样在代码段结束之后，即AutoReleasing代码块结束之后，就会释放，将self.person指向的对象的引用计数-1，从而达到内存管理，避免内存泄露的问题。当然，MRC中的代码也可以这样写

	{% codeblock Copy示例代码 lang:objc %}
		Person *person = [[Person alloc] init]; self.person = person; [person release];
	{% endcodeblock %}


与上面这段MRC的代码对应的ARC的代码应该是

	{% codeblock Copy示例代码 lang:objc %}
		Person *person = [[Person alloc] init];    self.person = person;
	{% endcodeblock %}

其实通过上面的ARC 与 MRC的代码对比我们也可以看到，LLVM将ARC代码转化成MRC代码的实现原理，使得开发者不需要关心内存管理的问题。

如果此时,Person类的copyWithZone 不返回自己的话,我们看一下如何实现深复制的功能

	{% codeblock Person类代码 lang:objc %}
		@interface Person : NSObject<NSCopying>

		@property (nonatomic, assign) BOOL isCoder;
		@property (nonatomic, assign) NSInteger age;
		@property (nonatomic, copy) NSString *name;

		@end

		@implementation Person

		- (instancetype)copyWithZone:(NSZone *)zone {
		    Person *person = [[Person alloc] init];
		    person.isCoder = self.isCoder;
		    person.age = self.age;
		    person.name = self.name;
		    
		    return person;
		}
		@end
	{% endcodeblock %}

如果Person实现了深复制的功能的话,那么上面的测试用例打印出来的就不相同了.同时引用计数也不会增加,而是两个Person对象的引用计数分别都为 **1**.

注:如果要实现一个对象的深复制的话,那么对象的修饰符一定要是copy才可以.如果copyWitZone返回self,那么与strong修饰符作用完全一样.

# 四、weak 属性修饰符简介

紧跟着上面的实现,我们在World中有Person的引用，在Person中也增加一个World的引用，来表示这个人属于哪个世界，如果这两个对象都是用strong修饰，则会存在内存泄露的情况,

如果这两个copyWithZone对象存在浅复制的情况则会造成内存泄露。用代码验证一下：

	
	{% codeblock Person类代码 lang:objc %}
		@interface Person : NSObject<NSCopying>

		@property (nonatomic, assign) BOOL isCoder;
		@property (nonatomic, assign) NSInteger age;
		@property (nonatomic, copy) NSString *name;
		@property (nonatomic, strong) World *world;

		@end

		@implementation Person

		- (instancetype)copyWithZone:(NSZone *)zone {
		    Person *person = [[Person alloc] init];
		    person.isCoder = self.isCoder;
		    person.age = self.age;
		    person.name = self.name;
		    
		    return person;
		}
		@end
	{% endcodeblock %}

	{% codeblock Person类代码 lang:objc %}
		//测试代码
		world = [[World alloc] init];
		person = [[Person alloc] init];
		person.name = @"terry";

		world.person =person;
		person.world = world;
	{% endcodeblock %}
	

上面代码中world引用了person,而person 同样引用了 world,当world调用release的时候,因为person还有引用,所以不会dealoc,而person也是同样的道理,这样就形成了强强引用的关系，造成内存泄露。所以，为了解决这种问题，IOS 中引入了weak的概念。

weak主要是用来解决两个对象之间相互引用造成的两个对象的内存无法释放的问题。

将Person中的world修饰符改为weak,即可解决问题循环引用的情况.

	{% codeblock Person类代码 lang:objc %}
		@interface Person : NSObject<NSCopying>

		@property (nonatomic, assign) BOOL isCoder;
		@property (nonatomic, assign) NSInteger age;
		@property (nonatomic, copy) NSString *name;
		@property (nonatomic, weak) World *world;

		@end

		@implementation Person

		- (instancetype)copyWithZone:(NSZone *)zone {
		    Person *person = [[Person alloc] init];
		    person.isCoder = self.isCoder;
		    person.age = self.age;
		    person.name = self.name;
		    
		    return person;
		}
		@end
	{% endcodeblock %}

那么，使用如下测试代码：

	{% codeblock 测试代码 lang:objc %}
		world = [[World alloc] init];
		person = [[Person alloc] init];
		person.name = @"terry";

		world.person =person;
		person.world = world;
	{% endcodeblock %}

代码执行完毕之后，person引用计数为2，world的引用计数为1(world自己还活着呢,1 就是表示的自己).

所以，当world释放的时候,person引用计数就会**减1**.当person对象释放的时候,发现就自己有对那块内存的引用,自然-1之后

引用计数变为0,执行dealloc方法,内存全部释放.

而这时候strong修饰符的使用也就没什么好说的了，被strong 修饰的对象，引用计数+1.

# **五、在写博客过程中发现的问题**

## 类函数与成员函数的区别

	第一点：写法不同，这种很明显的区别显而易见。
	第二点：内存管理方面的区别,比如

	{% codeblock 测试代码 lang:objc %}
		NSArray*array = [NSArray array];
		NSArray *array = [[NSArray alloc] init];
	{% endcodeblock %}

	记得前面也有提到过MRC 与ARC的转换，下面看这段代码就了解了类函数 与 成员函数的区别了。

	在MRC中需要这么写如下代码，否者会造成内存泄露

	{% codeblock 测试代码 lang:objc %}
		NSArray *array = [[[NSArray alloc] init] autoreleasing];
		//也可以这样写：
		NSArray *array = [[NSArray alloc] init]; 
		/*其他代码*/
		[array release];
		//但是如果使用Foundation中的类方法创建对象的话，则不需要上面那样写
		NSArray *array = [NSArray array];
	{% endcodeblock %}
	
这就是两者之间的区别。自己细细的领悟哦。

# **后序**

记录一下关于内存管理的自己的一些认知，如果有什么不对的，欢迎批评指正，大家共同进步。后续也会增加对于IOS 各方面的研究博客。欢迎 关注。