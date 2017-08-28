---
layout: post
title: Spring中定时任务用法
date: 2017-08-28 23:37:37
tags: [Spring, 定时任务]
---

在一些系统开发过程中, 难免会有定时任务去做一些事情完善系统的功能.本篇主要说明在Spring 中如何使用定时调度任务。
通过配置注解@EnableScheduling 来开启对定时任务的支持, 然后在要执行的函数上注解@Scheduled来声明这是一个定时任务。

## 创建一个计划任务的配置类
	
	package com.terrylmay;

	import org.springframework.context.annotation.Configuration;
	import org.springframework.scheduling.annotation.EnableScheduling;
	import org.springframework.stereotype.Component;

	@Configuration
	@Component("com.terrylmay")
	@EnableScheduling
	public class ScheduleConfig {

	}


## 创建计划任务, 并使用@Scheduled来注解

	package com.terrylmay;

	import java.util.Date;

	import org.springframework.scheduling.annotation.Scheduled;
	import org.springframework.stereotype.Service;

	@Service
	public class ScheduleTaskService {

		static int index = 0;

		@Scheduled(fixedRate = 5000)
		public void increaseIndex() {
			System.out.println(new Date() + ":" + index);
		}
		
		@Scheduled(cron = "0 38 23 ? * *")
		public void fixTime() {
			System.out.println(new Date() + ":" + "准时打印");
		}

	}


## 开发启动程序, 开启进程

	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ScheduleApplication {
		public static void main(String[] args) throws InterruptedException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ScheduleConfig.class);

		}

	}


## 观察定时调度任务的执行

	Mon Aug 28 23:37:23 CST 2017:0
	Mon Aug 28 23:37:28 CST 2017:0
	Mon Aug 28 23:37:33 CST 2017:0
	Mon Aug 28 23:37:38 CST 2017:0
	Mon Aug 28 23:37:43 CST 2017:0
	Mon Aug 28 23:37:48 CST 2017:0
	Mon Aug 28 23:37:53 CST 2017:0
	Mon Aug 28 23:37:58 CST 2017:0
	Mon Aug 28 23:38:00 CST 2017:准时打印
	Mon Aug 28 23:38:03 CST 2017:0
	Mon Aug 28 23:38:08 CST 2017:0

可以看到每隔5s钟打印一次, 当23:38的时候打印出了固定时间点调度的任务

以上就是Spring中定时调度任务的使用