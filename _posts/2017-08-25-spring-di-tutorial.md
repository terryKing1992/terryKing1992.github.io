---
layout: post
title: Spring中依赖注入的简单使用
date: 2017-08-25 20:05:37
tags: [Spring, 依赖注入]
---

依赖注入指容器负责创建和维护对象的起来关系.Spring IoC容器负责创建Bean并能够使用注入的方式供其他的类使用, 通过组合的方式实现类之间的解耦。Spring可以使用xml、注解以及java配置的方式实现依赖注入。

因为xml实现依赖注入比较麻烦而且基本没有代码提示， 所以本篇主要记录较为简单的注解的方式以及 java代码配置的方式。

## 一、哪些注解能够声明类为Bean

一般针对不同层的代码, 使用不同的注解来完成Bean的创建是比较明智的; 注解一方面能够实现依赖注入 另一方面增加代码的可读性

	@Component 对于一些工具类或者组件性质的类, 使用该注解
	@Repository 一般对于数据库层的访问类, 使用该注解
	@Service 对于一些提供服务的类 使用该注解
	@Controller 对于控制层的类, 比如处理Http请求的类 使用该注解

## 二、需要使用Bean的时候使用哪些注解能够获取Bean

在使用Bean的类中, 如果想将对象从容器中取出来, 可以使用如下的注解完成:

	@Autowired 一般情况下使用该注解, spring提供的注解
	@Inject    JSR-330提供的注解
	@Resource JSR-250提供的注解

## 三、如何使用最小依赖完成依赖注入的实验

#### 1、使用IDE 生成一个maven工程, 比如Intellij 或者eclipse
#### 2、在pom.xml文件中加入相关的依赖

	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>4.3.9.RELEASE</version>
	</dependency>
	<!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-core</artifactId>
		<version>4.3.9.RELEASE</version>
	</dependency>

#### 3、创建Spring Bean配置类,主要告诉Spring容器去哪里加载Bean

	package com.terrylmay;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class BeanConfig {

	}

其中com.terrylmay 是我工程下根包名, 即我的所有声明的Bean全部都在该包名下面
@Configuration 告诉Spring 这是一个配置类

@ComponentScan("com.terrylmay") 告诉Spring 去该包名下查找所有的Bean并加载到容器中

#### 4、创建Bean类


	package com.terrylmay;

	import org.springframework.stereotype.Component;

	@Component
	public class UserComponent {
		public String getUserName() {
			return "terrylmay";
		}
	}

如前面所有, 如果要声明Bean有4种方式, @Component是最普通也是最没有具体意义的一种

#### 5、创建应用的入口类
	
	package com.terrylmay;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationStart {
		public static void main(String[] args) {
			AnnotationConfigApplicationContext configContext = new AnnotationConfigApplicationContext(BeanConfig.class);
			UserComponent userComponent = configContext.getBean(UserComponent.class);

			System.out.println(userComponent.getUserName());

			configContext.close();
		}
	}

上述代码中使用 AnnotationConfigApplicationContext 作为Spring容器, 将配置类信息传入构造函数.AnnotationConfigApplicationContext 从相应的包名下查找Bean并自动加载。

所以在后面的程序中, 我们可以直接使用UserComponent.class 来获取该类的实例 并且调用方法。

到这里基本上依赖注入的Spring代码最小单元实验就已经结束了。不过上面的代码是使用@Component来声明一个Bean, 同时我们也可以在配置类中使用@Bean来声明一个Bean,

下面我们看简单配置类代码:

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
	}

使用这种方式配置Bean, 那么UserComponent类就不需要使用@Component等注解来修饰也可以达到同样的效果


