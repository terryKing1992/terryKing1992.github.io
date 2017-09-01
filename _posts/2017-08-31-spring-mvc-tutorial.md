---
layout: post
title: Spring中MVC的使用
date: 2017-08-29 22:00:00
tags: [Spring, MVC]
---

SpringMVC在服务开发框架中已经具有举足轻重的地位。下面我们来一起看一下如何使用最小的、最少的配置来完成spring mvc的框架的搭建


#### 添加pom文件依赖

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.terrylmay</groupId>
		<artifactId>com.terrylmay</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>war</packaging>

		<name>com.terrylmay</name>
		<url>http://maven.apache.org</url>

		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<java.version>1.8</java.version>
			<spring.version>4.3.9.RELEASE</spring.version>
			<java.servlet.version>3.1.0</java.servlet.version>
		</properties>

		<dependencies>
			<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>${spring.version}</version>
			</dependency>

			<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>javax.servlet-api</artifactId>
				<version>${java.servlet.version}</version>
				<scope>provided</scope>
			</dependency>

			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>3.8.1</version>
				<scope>test</scope>
			</dependency>
		</dependencies>
	</project>

####  创建Spring-MVC 配置类

	package com.terrylmay;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.config.annotation.EnableWebMvc;

	@Configuration
	@ComponentScan("com.terrylmay")
	@EnableWebMvc
	public class AppMVCConfig {

	}

#### 创建Spring-MVC 的入口类

程序入口继承WebApplicationInitializer类, 并实现相关接口, 可以打包部署到tomcat中

	package com.terrylmay;

	import javax.servlet.ServletContext;
	import javax.servlet.ServletException;
	import javax.servlet.ServletRegistration.Dynamic;

	import org.springframework.web.WebApplicationInitializer;
	import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
	import org.springframework.web.servlet.DispatcherServlet;

	/**
	 * Hello world!
	 *
	 */
	public class App implements WebApplicationInitializer {

		public void onStartup(ServletContext servletContext) throws ServletException {
			AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
			context.register(AppMVCConfig.class);
			context.setServletContext(servletContext);

			Dynamic servlet = servletContext.addServlet("dispatcher", new DispatcherServlet(context));
			servlet.addMapping("/");
			servlet.setLoadOnStartup(1);
		}
	}

#### 创建HelloWorldController 提供一个最简单的服务

	package com.terrylmay;

	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	public class HelloController {

		@RequestMapping("/hello")
		public String sayHello() {
			return "Hello World";
		}
	}


#### 在pom.xml 文件中加入打包所用的插件

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.terrylmay</groupId>
		<artifactId>com.terrylmay</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>war</packaging>

		<name>com.terrylmay</name>
		<url>http://maven.apache.org</url>

		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<java.version>1.8</java.version>
			<spring.version>4.3.9.RELEASE</spring.version>
			<java.servlet.version>3.1.0</java.servlet.version>
		</properties>

		<dependencies>
			<!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
			<!-- https://mvnrepository.com/artifact/org.springframework/spring-web -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-web</artifactId>
				<version>4.3.9.RELEASE</version>
			</dependency>

			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-webmvc</artifactId>
				<version>4.3.9.RELEASE</version>
			</dependency>

			<!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->
			<dependency>
				<groupId>javax.servlet</groupId>
				<artifactId>javax.servlet-api</artifactId>
				<version>${java.servlet.version}</version>
				<scope>provided</scope>
			</dependency>

			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>3.8.1</version>
				<scope>test</scope>
			</dependency>
		</dependencies>

		<build>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>2.6</version>
					<configuration>
						<failOnMissingWebXml>false</failOnMissingWebXml>
					</configuration>
				</plugin>
			</plugins>
		</build>
	</project>


#### 打包好的war包放在target目录下面, 我们将该war包放入到tomcat的webapps

![运行结果](/assets/images/2017-09-01-spring-mvc-tomcat-running-snapshot.png)


