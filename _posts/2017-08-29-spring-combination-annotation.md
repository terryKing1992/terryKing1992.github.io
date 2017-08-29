---
layout: post
title: Spring中组合注解的用法
date: 2017-08-29 08:00:00
tags: [Spring, 组合注解]
---

在前面的文章案例中, 对于配置类的开发最少需要两个注解: @ComponentScan 以及 @Configuration, 每次都重复敲这两行代码或者还有@Enable*等代码总会很麻烦， 有没有一种方法可以将两个注解合并成一个注解, 在写代码的时候只写一个注解就完成两件事情呢?答案是可以的, 就是这篇提到的组合注解的概念。

组合注解就是新增一个注解, 但是这个注解又被其他的元注解修饰。

下面我们来看一下怎么样来实现@ComponentScan 和 @Configuration的组合注解;

#### 创建一个注解CombinationAnnotation

	package com.terrylmay;

	import java.lang.annotation.Documented;
	import java.lang.annotation.ElementType;
	import java.lang.annotation.Retention;
	import java.lang.annotation.RetentionPolicy;
	import java.lang.annotation.Target;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@ComponentScan
	@Configuration
	public @interface CombinationAnnotation {
		String [] value() default {};
	}


#### 创建一个Bean, 用来验证组合注解是否生效

	package com.terrylmay;

	import java.util.Date;

	import org.springframework.stereotype.Service;

	@Service
	public class ScheduleTaskService {

		public void increaseIndex() {
			System.out.println(new Date());
		}
	}


#### 创建配置类, 使用组合注解修饰

	package com.terrylmay;

	@CombinationAnnotation("com.terrylmay")
	public class BeanConfig {

	}


#### 创建启动类

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfig.class);

			ScheduleTaskService taskService = context.getBean(ScheduleTaskService.class);
			taskService.increaseIndex();
			context.close();
		}
	}

#### 分析结果

	运行启动类可以发现, ScheduleTaskService中的increaseIndex方法已经打印出来了, 说明Bean已经正确加载, 继而说明组合注解已经生效。

