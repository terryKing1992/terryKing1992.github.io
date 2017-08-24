---
layout: post
title: Spring中EL表达式的使用与资源的访问
date: 2017-08-25 20:05:37
tags: [Spring, EL表达式, 资源访问]
---

Spring开发中总会遇到对于一些文件访问或者常量等数据的配置, 比如数据库访问URL, 驱动程序包名(com.mysql.driver.DriverManager类似的)等等, 如何将这些数据从properties文件中读入到程序中?除了自己写properties文件读取外, 还可以使用EL表达式来完成该项工作。

本篇博客主要记录如何使用EL表达式来实现资源的注入功能

## 一、普通字符串的注入

1、首先我们创建配置配ResourceConfig
	
	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class ResourceConfig {

		@Value("terrylmay")
		String userName;

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

	}

2、创建系统启动类

	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			// 首先我们获取ResourceConfig的实例
			ResourceConfig resourceConfig = context.getBean(ResourceConfig.class);
			System.out.println(resourceConfig.getUserName());
			context.close();
		}
	}

通过打印, 我们就可以看到控制台已经打印出来了ResourceConfig.userName的注入值terrylmay了。使用字符串注入, 直接在@Value("XXX")里面填写要注入的字符串就可以了


## 二、操作系统属性注入

1、直接修改ResourceConfig, 增加操作系统属性注入, 这里的操作系统属性不包括系统bash_profile里面配置的环境变量
	
	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class ResourceConfig {

		// 字符串直接注入
		@Value("terrylmay")
		String userName;

		//操作系统属性注入
		@Value("#{systemProperties['os.name']}")
		String osName;

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

		public String getOsName() {
			return osName;
		}

		public void setOsName(String osName) {
			this.osName = osName;
		}

	}

2、修改ApplicationEntry类, 将注入的系统属性打印出来
	
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




## 三、注入EL表达式运算结果

	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class ResourceConfig {
		//字符串注入
		@Value("terrylmay")
		String userName;

		// 操作系统属性注入
		@Value("#{systemProperties['os.name']}")
		String osName;
		
		//EL 表达式求值
		@Value("#{ 2 * 100.0}")
		Integer addResult;

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

		public String getOsName() {
			return osName;
		}

		public void setOsName(String osName) {
			this.osName = osName;
		}

	}



## 四、注入Bean的属性

1、新建一个Bean, 命名为: User
	
	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.stereotype.Component;

	@Component
	public class User {

		@Value("terrylmay")
		String userName;

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

	}

2、修改启动类 打印注入结果
	
	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			// 首先我们获取ResourceConfig的实例
			ResourceConfig resourceConfig = context.getBean(ResourceConfig.class);
			System.out.println("字符串注入值:" + resourceConfig.getUserName());
			System.out.println("操作系统属性注入值:" + resourceConfig.getOsName());
			System.out.println("EL 表达式计算值:" + resourceConfig.addResult);
			System.out.println("注入Bean的属性值:" + resourceConfig.user);
			context.close();
		}
	}



## 五、注入文件内容

1、在com.terrylmay包名下创建一个文本文件text.properties, 内容为:
	
	book.author=terry
	book.date=2017-09-12

2、修改ResourceConfig文件, 加入文件内容注入

	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.core.io.Resource;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class ResourceConfig {
		// 字符串注入
		@Value("terrylmay")
		String userName;

		// 操作系统属性注入
		@Value("#{systemProperties['os.name']}")
		String osName;

		// EL 表达式求值
		@Value("#{ 2 * 100.0}")
		Integer addResult;

		// Bean 属性值注入
		@Value("#{ user.userName }")
		String user;

		// 网站Resource注入
		@Value("https://www.baidu.com")
		Resource url;

		@Value("classpath:com/terrylmay/text.properties")
		Resource propertiesFile;

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

		public String getOsName() {
			return osName;
		}

		public void setOsName(String osName) {
			this.osName = osName;
		}

	}

3、修改程序入口文件, 将注入文件内容打印出来

	package com.terrylmay;

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStreamReader;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			// 首先我们获取ResourceConfig的实例
			ResourceConfig resourceConfig = context.getBean(ResourceConfig.class);
			System.out.println("字符串注入值:" + resourceConfig.getUserName());
			System.out.println("操作系统属性注入值:" + resourceConfig.getOsName());
			System.out.println("EL 表达式计算值:" + resourceConfig.addResult);
			System.out.println("注入Bean的属性值:" + resourceConfig.user);
			System.out.println("注入网址资源值:" + resourceConfig.url);
			BufferedReader reader = new BufferedReader(
					new InputStreamReader(resourceConfig.propertiesFile.getInputStream()));
			String line = null;
			while ((line = reader.readLine()) != null) {
				System.out.println("注入文件内容值:" + line);
			}
			context.close();
		}
	}


## 六、注入网址

1、修改Config文件, 加入网址注入

	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.core.io.Resource;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class ResourceConfig {
		// 字符串注入
		@Value("terrylmay")
		String userName;

		// 操作系统属性注入
		@Value("#{systemProperties['os.name']}")
		String osName;

		// EL 表达式求值
		@Value("#{ 2 * 100.0}")
		Integer addResult;
		
		//Bean 属性值注入
		@Value("#{ user.userName }")
		String user;
		
		//网站Resource注入
		@Value("https://www.baidu.com")
		Resource url;
		
		

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

		public String getOsName() {
			return osName;
		}

		public void setOsName(String osName) {
			this.osName = osName;
		}

	}

2、修改入口程序, 打印结果

	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			// 首先我们获取ResourceConfig的实例
			ResourceConfig resourceConfig = context.getBean(ResourceConfig.class);
			System.out.println("字符串注入值:" + resourceConfig.getUserName());
			System.out.println("操作系统属性注入值:" + resourceConfig.getOsName());
			System.out.println("EL 表达式计算值:" + resourceConfig.addResult);
			System.out.println("注入Bean的属性值:" + resourceConfig.user);
			System.out.println("注入网址资源值:" + resourceConfig.url);
			context.close();
		}
	}

## 七、注入属性文件

1、注入属性文件, 首先需要声明属性文件在哪里, 将如下代码修饰ResourceConfig类

	@PropertySource("classpath:com/terrylmay/text.properties")

2、添加注入的值

	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Value;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.core.io.Resource;

	@Configuration
	@ComponentScan("com.terrylmay")
	@PropertySource("classpath:com/terrylmay/text.properties")
	public class ResourceConfig {
		// 字符串注入
		@Value("terrylmay")
		String userName;

		// 操作系统属性注入
		@Value("#{systemProperties['os.name']}")
		String osName;

		// EL 表达式求值
		@Value("#{ 2 * 100.0}")
		Integer addResult;

		// Bean 属性值注入
		@Value("#{ user.userName }")
		String user;

		// 网站Resource注入
		@Value("https://www.baidu.com")
		Resource url;

		@Value("classpath:com/terrylmay/text.properties")
		Resource propertiesFile;
		
		//属性文件注入
		@Value("${book.author}")
		String author;

		//属性文件注入
		@Value("${book.date}")
		String date;

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

		public String getOsName() {
			return osName;
		}

		public void setOsName(String osName) {
			this.osName = osName;
		}

	}

3、将注入的值打印出来
	
	package com.terrylmay;

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStreamReader;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ResourceConfig.class);
			// 首先我们获取ResourceConfig的实例
			ResourceConfig resourceConfig = context.getBean(ResourceConfig.class);
			System.out.println("字符串注入值:" + resourceConfig.getUserName());
			System.out.println("操作系统属性注入值:" + resourceConfig.getOsName());
			System.out.println("EL 表达式计算值:" + resourceConfig.addResult);
			System.out.println("注入Bean的属性值:" + resourceConfig.user);
			System.out.println("注入网址资源值:" + resourceConfig.url);
			BufferedReader reader = new BufferedReader(
					new InputStreamReader(resourceConfig.propertiesFile.getInputStream()));
			String line = null;
			while ((line = reader.readLine()) != null) {
				System.out.println("注入文件内容值:" + line);
			}
			
			System.out.println("注入属性文件值:" + resourceConfig.author);
			System.out.println("注入属性文件值:" + resourceConfig.date);
			context.close();
		}
	}


## 最后的打印结果为:

	字符串注入值:terrylmay
	操作系统属性注入值:Mac OS X
	EL 表达式计算值:200
	注入Bean的属性值:terrylmay
	注入网址资源值:URL [https://www.baidu.com]
	注入文件内容值:book.author=terry
	注入文件内容值:book.date=2017-09-12
	注入属性文件值:terry
	注入属性文件值:2017-09-12

上面列出的就是基本常见的EL 表达式 以及 资源的注入方式