---
layout: post
title: Spring中多线程用法
date: 2017-08-26 23:37:37
tags: [Spring, 多线程]
---

Spring通过任务执行器TaskExecutor来实现多线程与并发, 使用ThreadPoolTaskExecutor实现一个带有线程池的TaskExecutor。如果要开启对Spring异步任务特性的支持, 需要在配置类中使用@EnableAsync来修饰, 并通过在时机执行的Bean方法中使用@Async注解来声明某方法是一个异步任务

## 开发一个配置类, 使用@EnableAsync注解

	package com.terrylmay;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.scheduling.annotation.EnableAsync;

	@Configuration
	@ComponentScan("com.terrylmay")
	@EnableAsync
	public class AsyncConfig {

	}

## 开发一个任务类, 使用@Async注解来声明某方法是一个异步任务

	package com.terrylmay;

	import org.springframework.scheduling.annotation.Async;
	import org.springframework.stereotype.Service;

	@Service
	public class CountService {

		@Async
		public void count() {
			System.out.println(Thread.currentThread());
		}

		@Async
		public void countOne() {
			System.out.println(Thread.currentThread());
		}
	}



在该类中, 我们打印出当前函数运行的线程信息, 如果是Main主线程 那么说明该函数不是异步任务
如果该方法不是在主线程中运行, 那么说明该任务是一个异步任务

## 开发入口启动类

	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class AsyncApplication {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AsyncConfig.class);
			CountService countService = context.getBean(CountService.class);
			countService.count();
			countService.countOne();
			System.out.println(Thread.currentThread());
			context.close();
		}
	}

打印信息为:
	Thread[main,5,main]
	Thread[SimpleAsyncTaskExecutor-1,5,main]
	Thread[SimpleAsyncTaskExecutor-1,5,main]

我们先看一下Thread的toString方法是怎么实现的

	 /**
     * Returns a string representation of this thread, including the
     * thread's name, priority, and thread group.
     *
     * @return  a string representation of this thread.
     */
    public String toString() {
        ThreadGroup group = getThreadGroup();
        if (group != null) {
            return "Thread[" + getName() + "," + getPriority() + "," +
                           group.getName() + "]";
        } else {
            return "Thread[" + getName() + "," + getPriority() + "," +
                            "" + "]";
        }
    }

由此可以看出counterService.count()方法是一个异步任务, 运行在SimpleAsyncTaskExecutor-1这个线程中, 优先级为5, 所属的群组为Main


## 如果我们配置一个异步任务的线程池, 打印结果又会是怎样的呢?

我们需要修改配置类, 在其中加入TaskExecutor 的Bean

	package com.terrylmay;

	import java.util.concurrent.Executor;

	import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.scheduling.annotation.AsyncConfigurer;
	import org.springframework.scheduling.annotation.EnableAsync;
	import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

	@Configuration
	@ComponentScan("com.terrylmay")
	@EnableAsync
	public class AsyncConfig implements AsyncConfigurer {

		public Executor getAsyncExecutor() {
			ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
			taskExecutor.setMaxPoolSize(100);
			taskExecutor.setCorePoolSize(20);
			taskExecutor.setQueueCapacity(100);
			taskExecutor.initialize();
			return taskExecutor;
		}

		public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
			// TODO Auto-generated method stub
			return null;
		}
	}


这时, 我们再运行Main函数, 会发现打印线程名称已经变了

	Thread[main,5,main]
	Thread[ThreadPoolTaskExecutor-2,5,main]
	Thread[ThreadPoolTaskExecutor-1,5,main]

已经从原来的每次New一个线程运行 切换到了从线程池里面获取线程来运行




