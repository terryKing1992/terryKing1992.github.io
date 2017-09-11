---
layout: post
title: 快速搭建SpringBoot框架
date: 2017-09-11 23:00:00
tags: [SpringBoot]
---

SpringBoot核心功能包括能够使用jar包独立运行、内嵌Servlet容器、提供starter方式简化maven配置、自动配置Spring、无xml配置。正是因为这些特性 才使得SpringBoot能够快速构建项目、极大的提高了开发、部署效率同时与云计算天然集成。

## SpringBoot快速搭建

### 首先我们创建一个Maven项目, 并将SpringBoot依赖加入到pom.xml文件中

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.terrylmay</groupId>
		<artifactId>springboot</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>jar</packaging>

		<name>springboot</name>
		<url>http://maven.apache.org</url>

		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
			<springboot.version>1.5.6.RELEASE</springboot.version>
		</properties>

		<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->


		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-web</artifactId>
				<version>${springboot.version}</version>
			</dependency>

			<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test -->
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-test</artifactId>
				<version>1.5.6.RELEASE</version>
				<scope>test</scope>
			</dependency>

		</dependencies>
		<build>
			<plugins>
				<plugin>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-maven-plugin</artifactId>
				</plugin>
			</plugins>
		</build>
	</project>

我们需要把原来建立Maven项目时自带的junit依赖删除, 因为SpringBoot Test中已经包含了该插件。
这里我们增加了Spring Boot对Web的支持, 并且加入了SpringBoot对于Test的支持 以及 编译插件, 通过刷新依赖我们可以看到依赖树大概是如下这个样子:

![SpringBoot Web依赖](/assets/images/2017-09-11-spring-boot-maven-web-dependencies.png)

### 新建SpringBoot的入口类, 并且在入口类中加入Restful接口

	package com.terrylmay.springboot;

	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RestController;

	/**
	 * Hello world!
	 *
	 */
	@SpringBootApplication
	@RestController
	public class App {
		public static void main(String[] args) {
			SpringApplication.run(App.class, args);
		}

		@RequestMapping(value = "/sayHello", method = RequestMethod.GET)
		public String sayHello() {
			return "HelloWorld";
		}
	}

这里我们创建了一个应用, 该应用中有一个restful接口 返回了HelloWorld字符串
我们右键以JavaApplication模式运行, 可以看到类似的如下输出:

	
	  .   ____          _            __ _ _
	 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
	( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
	 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
	  '  |____| .__|_| |_|_| |_\__, | / / / /
	 =========|_|==============|___/=/_/_/_/
	 :: Spring Boot ::        (v1.5.6.RELEASE)

	2017-09-11 21:10:45.049  INFO 1603 --- [           main] com.terrylmay.springboot.App             : Starting App on terrydeMacBook-Air.local with PID 1603 (/Users/terrylmay/Github/spring-tutorial/springboot/target/classes started by terrylmay in /Users/terrylmay/Github/spring-tutorial/springboot)
	2017-09-11 21:10:45.054  INFO 1603 --- [           main] com.terrylmay.springboot.App             : No active profile set, falling back to default profiles: default
	2017-09-11 21:10:45.201  INFO 1603 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5e955596: startup date [Mon Sep 11 21:10:45 CST 2017]; root of context hierarchy
	2017-09-11 21:10:47.527  INFO 1603 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
	2017-09-11 21:10:47.554  INFO 1603 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
	2017-09-11 21:10:47.556  INFO 1603 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.16
	2017-09-11 21:10:47.754  INFO 1603 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
	2017-09-11 21:10:47.754  INFO 1603 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2557 ms
	2017-09-11 21:10:47.982  INFO 1603 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
	2017-09-11 21:10:47.989  INFO 1603 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
	2017-09-11 21:10:47.990  INFO 1603 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
	2017-09-11 21:10:47.990  INFO 1603 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
	2017-09-11 21:10:47.991  INFO 1603 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
	2017-09-11 21:10:48.496  INFO 1603 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5e955596: startup date [Mon Sep 11 21:10:45 CST 2017]; root of context hierarchy
	2017-09-11 21:10:48.627  INFO 1603 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/sayHello],methods=[GET]}" onto public java.lang.String com.terrylmay.springboot.App.sayHello()
	2017-09-11 21:10:48.633  INFO 1603 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
	2017-09-11 21:10:48.634  INFO 1603 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
	2017-09-11 21:10:48.684  INFO 1603 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
	2017-09-11 21:10:48.684  INFO 1603 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
	2017-09-11 21:10:48.745  INFO 1603 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
	2017-09-11 21:10:49.001  INFO 1603 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
	2017-09-11 21:10:49.132  INFO 1603 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
	2017-09-11 21:10:49.157  INFO 1603 --- [           main] com.terrylmay.springboot.App             : Started App in 4.695 seconds (JVM running for 5.279)
	2017-09-11 21:11:02.630  INFO 1603 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet 'dispatcherServlet'
	2017-09-11 21:11:02.630  INFO 1603 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization started
	2017-09-11 21:11:02.654  INFO 1603 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : FrameworkServlet 'dispatcherServlet': initialization completed in 24 ms

从日志中我们可以看到, SpringBoot对Servlet容器进行了一系列的配置, 并且配置了访问路径帮我们做了dispatcherServlet的配置, 同时在8080端口启动tomcat容器并将应用运行起来

我们通过 http://localhost:8080/sayHello即可访问程序中的restful接口. 搭建一个应用就是这么容易


