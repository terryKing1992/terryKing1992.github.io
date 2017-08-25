---
layout: post
title: Spring中Profile的使用
date: 2017-08-26 11:06:37
tags: [Spring, Profile]
---

在应用分环境启动的场景中, 我们会碰到对于测试环境的数据库 与 生产环境的数据库(当然也有可能是其他的资源配置)是不同的情况, 那么我们就需要对于这种情况来获取不同的数据库访问字符串, 这时候我们就可以通过Profile注解来完成这项工作

下面我们看一下profile怎么解决不同环境下数据库访问字符串以及用户名密码不一致的情况

## Profile区分不同环境资源

#### 创建数据库资源的接口类

	package com.terrylmay;

	public interface IDBResourceConfig {
		public String getDBURL();

		public String getDBUserName();

		public String getDBPassword();
	}

#### 创建Dev环境的配置类, 实现接口类

	package com.terrylmay;

	import org.springframework.context.annotation.Profile;
	import org.springframework.stereotype.Component;

	@Component
	@Profile("dev")
	public class DevDBResourceConfig implements IDBResourceConfig {

		public String getDBURL() {
			return "127.0.0.1:3306/test";
		}

		public String getDBUserName() {
			return "db_admin_for_dev";
		}

		public String getDBPassword() {
			return "db_password_for_dev";
		}

	}

#### 创建Production环境的配置类, 并实现IDBResourceConfig接口

	package com.terrylmay;

	import org.springframework.context.annotation.Profile;
	import org.springframework.stereotype.Component;

	@Component
	@Profile("prod")
	public class ProdDBResourceConfig implements IDBResourceConfig {

		public String getDBURL() {
			return "10.60.11.121:3306/test";
		}

		public String getDBUserName() {
			return "db_admin_for_prod";
		}

		public String getDBPassword() {
			return "db_password_for_prod";
		}

	}

#### 创建启动类, 使用context进行环境的配置

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
			context.getEnvironment().setActiveProfiles("dev");
			context.register(ResourceConfig.class);
			context.refresh();
			IDBResourceConfig dbConfig = context.getBean(IDBResourceConfig.class);
			System.out.println(dbConfig.getDBURL());
			context.close();
		}
	}

可以看到程序运行之后打印出来了127.0.0.1：3306的信息, 如果修改代码变为setActiveProfiles("prod"), 那么便会打印出生产环境的相关信息

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
			context.getEnvironment().setActiveProfiles("prod");
			context.register(ResourceConfig.class);
			context.refresh();
			IDBResourceConfig dbConfig = context.getBean(IDBResourceConfig.class);
			System.out.println(dbConfig.getDBURL());
			context.close();
		}
	}

可以看到打印出了10.60.11.121:3306/test信息, 该启动程序中 与 其他的启动程序不太一样, 该启动程序是先创建了容器, 然后设置环境, 然后在设置配置类, 最后才获取Bean的信息并验证. 如果先设置配置类的话, 程序会报Bean未定义的错误。

以上的程序还是可以优化一下:

优化1、在定义数据库Bean的时候, 也可以引入前面我们提到的@PropertySource 使用@Value 来进行属性文件变量的注入。 这样我们定义2个数据库配置文件也能完成相关的操作.

优化2、启动程序的时候通过Shell脚本启动, 同时在Shell脚本中向main函数中传入一些相关的参数比如dev|prod等配置项, 这样就能完全做到同一个包仅仅通过脚本的启动参数的不同来部署到不同的环境中。



