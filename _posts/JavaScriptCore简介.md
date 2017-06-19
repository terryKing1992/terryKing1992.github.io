---
title: JavaScriptCore简介
date: 2016-02-21 11:16:26
tags: [JavascriptCore, 执行引擎]
---

##**到底Javascript core是什么**

前端开发的同学应该知道，浏览器核心模块主要是渲染引擎和JavaScript引擎两部分组成。前者用于处理页面布局，渲染及DOM结构等，后者用于JavaScript的解析、执行及DOM交互等。
JavaScriptCore是一种JavaScript引擎，主要为webkit提供脚本处理能力（其主要以safari浏览器为代表）。JavaScriptCore是开源webkit的一部分。

<!-- more -->

##苹果为什么要在native应用中（非webview）引入JavaScript

首先，脱离了浏览器外壳和繁重的UI布局和渲染的JavaScript引擎，无疑可以将JavaScript的能力更轻便地、高性能地带给原生的iOS应用，给应用开发者提供更多的想象力（无论是PC还是移动的浏览器，UI和GUI部分都是最重要的性能瓶颈和优化点）。

其次，在移动开发场景中确实有众多的开发者对JavaScript有需求。在google上搜索”embed javascript engine ios”可以得到大量的实践和博文。

再次，苹果已经在mac上得到很不错的实践和反响。引入JavaScriptCore即扩大了SDK能力又讨好了开发者，何乐不为呢。

##**Javascript Core能做什么**

	1、OC 调用 JS代码
	2、JS代码 调用 OC 代码

##Javascript中有哪些必须知道的类

在开始使用Javascript Core编程之前，需要了解该框架中的几个主类。

	#ifndef JavaScriptCore_h
	#define JavaScriptCore_h

	#include <JavaScriptCore/JavaScript.h>
	#include <JavaScriptCore/JSStringRefCF.h>

	#if defined(__OBJC__) && JSC_OBJC_API_ENABLED

	#import "JSContext.h"
	#import "JSValue.h"
	#import "JSManagedValue.h"
	#import "JSVirtualMachine.h"
	#import "JSExport.h"

	#endif

	#endif /* JavaScriptCore_h */

JSContext: JS代码的上下文，主要保持JS 中的一些变量 以及方法。
JSValue: js 变量或者方法 在 原生里面对应的对象。
JSManagedValue：防止强引用JSValue可能会造成内存泄露，使用JSManagedValue能够有效的管理JSValue的内存。(目前我就知道这个功能)
JSVirtualMachine：JS代码的执行环境
JSExport：用来告诉JSContext 原生提供哪些JSAPI 供JS 使用。

	##JS对象 与OC 对象之间的映射关系又是怎样的呢？
	  Objective-C type  |   JavaScript type
	--------------------+---------------------
	        nil         |     undefined
	       NSNull       |        null
	      NSString      |       string
	      NSNumber      |   number, boolean
	    NSDictionary    |   Object object
	      NSArray       |    Array object
	       NSDate       |     Date object
	      NSBlock (1)   |   Function object (1)
	         id (2)     |   Wrapper object (2)
	       Class (3)    | Constructor object (3)
	##在OC代码中我们应该怎么使用Javascript Core
	JSContext *context = [[JSContext alloc] init];

	NSString *javaScript = @"var sum = 1 + 2;";
	[context evaluateScript:javaScript];

	JSValue *sum = context[@"sum"];
	NSLog(@"OC 执行JS 代码结果:%@", sum);
	JS 调用OC代码：
	 JSContext *context = [[JSContext alloc] init];

	//在OC中实现对应的JS函数log，并且加入到JS上下文中
	context[@"log"] = ^(JSValue *value) {
	    NSLog(@"%@", [value toString]);
	};

	NSString *javaScript = @"log('我是原生的函数打印出来的')";
	[context evaluateScript:javaScript];

JS 调用OC代码 其实分几步进行：

	- 由原生实现log函数，并且加入到JS的上下文中；
	- 写JS代码 调用 原生的log函数
	- 有JSContext 执行 JS代码。只不过在原生执行JS代码的时候，JS又调用了原生的函数而已。

##**如何实现 面向对象的JS调用**

加入我们要在JS中这么写代码调用原生函数console.log(params)原生应该怎么实现；

	//
	//  JSExecutor.h
	//  JavascriptCore
	//
	//  Created by terry on 15/12/19.
	//

	#import <Foundation/Foundation.h>
	#import <JavaScriptCore/JavaScriptCore.h>

	@protocol BaseJSAPI<JSExport>
	- (void)log:(JSValue *)params;
	@end

	@interface JSExecutor : NSObject<BaseJSAPI>
	@property (nonatomic, strong) JSContext *context;

	- (JSValue *)evalueScript:(NSString *)script;
	- (JSValue *)jsValueForKey:(NSString *)key;
	@end

自己在对JSContext的函数封装一层，并且定义需要export出去的API，比如头文件BaseJSAPI中的log方法，同时，另外一个类实现这个方法。并将自己想要的console加入到context中。

	//
	//  JSExecutor.m
	//  JavascriptCore
	//
	//  Created by terry on 15/12/19.
	//

	#import "JSExecutor.h"

	@implementation JSExecutor
	- (instancetype)init {
	    self = [super init];

	    if (self) {
	        _context = [[JSContext alloc] init];
	        [self initJSContextMechine];
	    }
	    return self;
	}

	- (void)initJSContextMechine {
	    __weak typeof (self) weakSelf = self;

	    _context[@"console"] = weakSelf;
	}

	- (void)log:(JSValue *)params {
	    NSLog(@"%@", [params toString]);
	}

	- (JSValue *)evalueScript:(NSString *)script {
	    return [_context evaluateScript:script];
	}

	- (JSValue *)jsValueForKey:(NSString *)key {
	    return _context[key];
	}
	@end

这样当我们在JS中调用OC代码打印的时候就可以这么写了，console.log(“123”);

	JSExecutor *executor = [[JSExecutor alloc] init];

	NSString *javaScript = @"console.log('我是JSExecutor中提供的log函数打印出来的')";
	[executor evalueScript:javaScript];

因为JSExecutor实现了JSExport协议，在executor初始化的时候也初始化了JSContext，而JSContext 会把该executor对象对应的类 加入到JSContext的JSWrapperMap。这样，在JS引擎在看到console的时候，会找到executor这个对象，然后再执行executor.log();这样就相当于在原生实现了面向对象的API 提供。

如果想定义其他的JS方法，只需要在BaseJSAPI中增加新的方法，并且在JSExecutor 中实现即可。