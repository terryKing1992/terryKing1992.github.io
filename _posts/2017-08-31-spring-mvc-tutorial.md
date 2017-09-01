---
layout: post
title: Spring中MVC的使用
date: 2017-08-29 22:00:00
tags: [Spring, MVC]
---

SpringMVC在服务开发框架中已经具有举足轻重的地位。下面我们来一起看一下如何使用最小的、最少的配置来完成spring mvc的框架的搭建 加上 使用logback来配置日志的输出


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


#### 使用Logback配置日志模块

1、在pom.xml文件中增加logback日志依赖

	<!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-classic -->
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.2.3</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-core -->
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-core</artifactId>
		<version>1.2.3</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>1.7.25</version>
	</dependency>

添加了日志依赖之后, 我们可以看到在ReferenceLibrary中多了几个jar包依赖

![Logback Jar 包依赖](/assets/images/2017-09-01-spring-logback-dependencies.png)

2、配置logback.xml 文件, 配置日志输出格式、输出Console 还是 日志文件等
	
	<?xml version="1.0" encoding="UTF-8"?>
	<configuration scan="true" scanPeriod="60 seconds">
		<property name="LOG_HOME" value="/Users/terrylmay/logs" />
		<!-- Simple file output -->
		<appender name="FILE"
			class="ch.qos.logback.core.rolling.RollingFileAppender">
			<!-- encoder defaults to ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
			<encoder>
				<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} -
					%msg%n</pattern>
				<charset>UTF-8</charset> <!-- 此处设置字符集 -->
			</encoder>
			<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
				<!-- rollover daily 配置日志所生成的目录以及生成文件名的规则 -->
				<fileNamePattern>${LOG_HOME}/log_%d{yyyyMMdd}.%i.log
				</fileNamePattern>
				<timeBasedFileNamingAndTriggeringPolicy
					class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
					<!-- or whenever the file size reaches 64 MB -->
					<maxFileSize>64 MB</maxFileSize>
				</timeBasedFileNamingAndTriggeringPolicy>
				<maxHistory>30</maxHistory>
			</rollingPolicy>


			<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
				<level>INFO</level>
			</filter>
			<!-- Safely log to the same file from multiple JVMs. Degrades performance! -->
			<prudent>false</prudent>
		</appender>


		<!-- Console output -->
		<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
			<!-- encoder defaults to ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
			<encoder>
				<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} -
					%msg%n</pattern>
				<charset>UTF-8</charset> <!-- 此处设置字符集 -->
			</encoder>
			<!-- Only log level WARN and above -->
			<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
				<level>INFO</level>
			</filter>
		</appender>

		<root level="INFO">
			<appender-ref ref="FILE" />
			<appender-ref ref="STDOUT" />
		</root>
	</configuration>
	

3、在HelloController类中使用日志输出

	package com.terrylmay;

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	public class HelloController {

		private static final Logger LOG = LoggerFactory.getLogger(HelloController.class);

		@RequestMapping("/hello")
		public String sayHello() {
			LOG.info("Hello 接口被调用了");
			return "Hello World";
		}
	}

4、打包运行查看日志是否有输出

![Logback模块作用之后的输出](/assets/images/2017-09-01-spring-logback-console.png)




