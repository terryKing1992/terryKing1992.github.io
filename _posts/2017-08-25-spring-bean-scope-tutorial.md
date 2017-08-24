---
layout: post
title: Spring中Bean的Scope
date: 2017-08-25 21:16:37
tags: [Spring, Bean Scope]
---

Bean的Scope指Spring容器是如何创建Bean实例的, Bean的创建方式使用@Scope来声明

1、Singleton: Spring默认创建Bean的方式, 整个容器全局共享一个实例
2、Prototype: 每次调用就会新建一个实例
3、Request: 在Web应用中 给每一个http Request创建一个Bean实例
4、Session: 在Web应用中 每一个Session会话创建一个Bean实例

基于上篇博客中的代码, 再新建一个@Scope("Prototype")的Bean类, 通过多次从容器中获取Bean查看是否是同一个对象

一、首先创建一个Bean类 命名为UserService

	package com.terrylmay;

	import org.springframework.context.annotation.Scope;
	import org.springframework.stereotype.Service;

	@Service
	@Scope("prototype")
	public class UserService {
		public String getUserName() {
			return "terrylmay";
		}
	}


二、修改上篇中的程序入口类

	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	/**
	 * SpringApplication entry
	 *
	 */
	public class ApplicationStart {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext configContext = new AnnotationConfigApplicationContext(BeanConfig.class);
			UserComponent userComponent = configContext.getBean(UserComponent.class);
			UserComponent userComponent1 = configContext.getBean(UserComponent.class);

			UserService userService = configContext.getBean(UserService.class);
			UserService userService1 = configContext.getBean(UserService.class);
			System.out.println(userComponent.getUserName());
			System.out.println(userService.getUserName());

			System.out.println(userComponent == userComponent1);
			System.out.println(userService == userService1);
			configContext.close();
		}
	}

最后程序的输出为:

	terrylmay
	terrylmay
	true
	false
	
表示Bean的默认scope容器中只产生一个实例, 而使用@Scope("prototype") 则每从容器中取出一次Bean 则会产生一个新的实例。