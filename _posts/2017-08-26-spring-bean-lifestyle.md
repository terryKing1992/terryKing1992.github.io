---
layout: post
title: Spring中Bean的初始化及销毁
date: 2017-08-25 22:35:37
tags: [Spring, 初始化, 销毁]
---

Bean在使用之前, 有时候需要做一些初始化的工作, 但是又由于Bean是交给容器托管的, 所以没办法在构造函数里面进行某些变量的初始化, 所以Spring 以及 Java提供了一种初始化后以及销毁前的一些操作。

实现托管的初始化工作有两种方式:

1、Java配置方式: 使用@Bean的initMethod 和 destroyMethod方法进行配置

2、JSR-250的@PostConstruct和@PreDestroy

## Java配置Bean的初始化以及销毁函数:

#### 创建Bean类, 但是不对Bean进行@Component等修饰符修饰

	package com.terrylmay;

	public class BeanInitAndDestroy {
		public void init() {
			System.out.println("Bean Init Involved");
		}

		public void destroy() {
			System.out.println("Bean Destroy Invalved");
		}
	}

#### 使用@Bean配置InitMethod 和 destroyMethod

	package com.terrylmay;

	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class BeanConfig {
		@Bean
		public UserComponent getUserComponent() {
			return new UserComponent();
		}

		@Bean(initMethod="init", destroyMethod="destroy")
		public BeanInitAndDestroy getBeanInit() {
			return new BeanInitAndDestroy();
		}
	}

#### 创建应用启动类

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			BeanInitAndDestroy beanInit = context.getBean(BeanInitAndDestroy.class);
			context.close();
		}
	}


这样在运行启动类的时候, 就可以看到 Bean Init Involved 打印出来了, 因为应用启动的时候会初始化所有的Bean类,并放到Spring容器中.自然会调用 BeanInitAndDestroy 的init方法. 当容器销毁的时候, BeanInitAndDestroy实例同样会调用destroy方法进行最后的一些资源释放等工作

## JSR-250配置Bean的初始化以及销毁工作


#### 创建Bean类

	package com.terrylmay;

	import javax.annotation.PostConstruct;
	import javax.annotation.PreDestroy;

	import org.springframework.stereotype.Component;

	@Component
	public class JSRBeanInitAndDestroy {

		@PostConstruct
		public void init() {
			System.out.println("JSRBeanInitAndDestroy Instance init");
		}

		@PreDestroy
		public void destroy() {
			System.out.println("JSRBeanInitAndDestroy Instance Destroy");
		}
	}

#### 在启动类中进行调用 与 释放

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			JSRBeanInitAndDestroy beanInit = context.getBean(JSRBeanInitAndDestroy.class);
			context.close();
		}
	}

运行启动类之后, 同样我们可以看到在类JSRBeanInitAndDestroy创建过程中的初始化调用以及Destroy调用, 打印出来的

	JSRBeanInitAndDestroy Instance init
	JSRBeanInitAndDestroy Instance Destroy




