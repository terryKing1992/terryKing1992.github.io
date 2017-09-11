---
layout: post
title: 快速搭建SpringBoot框架
date: 2017-09-11 23:00:00
tags: [SpringBoot]
---

上篇我们看到了如何开发一个最简单的SpringBoot应用, 在上篇中我们使用@SpringBootApplication注解来完成应用的注解, 像前面我们看到的SpringMVC的配置大概都需要配置@Configuration、@EnableWebMvc、@ComponentScan的配置, 但是在SpringBoot应用中只需要一个注解就可以完成上面的所有工作. 因为SpringBootApplication是一个组合注解.
下面我们就一起来看一下SpringBoot中的配置的基本使用。

### 简单花哨的Banner定制

#### 切换Banner图案

在SpringBoot中启动图案是一个Spring的图案

	
	  .   ____          _            __ _ _
	 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
	( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
	 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
	  '  |____| .__|_| |_|_| |_\__, | / / / /
	 =========|_|==============|___/=/_/_/_/
	 :: Spring Boot ::        (v1.5.6.RELEASE)

大家能够看出来这是一个Spring的标识, 我们可以改成这个样子的, 怎么修改呢?

		.____                        _____.___.             
	|    |    _______  __ ____   \__  |   | ____  __ __ 
	|    |   /  _ \  \/ // __ \   /   |   |/  _ \|  |  \
	|    |__(  <_> )   /\  ___/   \____   (  <_> )  |  /
	|_______ \____/ \_/  \___  >  / ______|\____/|____/ 
	        \/               \/   \/                       

我们在src/main/resources目录下创建一个banner.txt的文件, 通过http://patorjk.com/software/taag 网站生成字符, 并将网站生成的字符复制到banner.txt中

再次运行程序就可以看到我们刚才生成的字符作为Banner了

#### 关闭banner图案

在代码中使用即可关闭Banner的输出

	public static void main(String[] args) {
		SpringApplication application = new SpringApplication(App.class);
		application.setBannerMode(Banner.Mode.OFF);
		application.run(args);
	}

### SpringBoot中的运行时参数配置文件

SpringBoot中会默认去读取application.properties中的配置, 如果没有该文件或者该文件中没有相关的配置, 那么SpringBoot会使用自带的默认配置启动应用.

#### 在src/main/resources目录下创建application.properties, 来看一下应用的一些基本配置

	server.contextPath=/springboot # 设置应用的上下文根, 如果设置了那么访问的时候需要加上该上下文根, 比如访问sayHello接口的时候, 需要使用http://localhost:9080/springboot/sayHello
	server.port=9080 # tomcat在9080端口上面启动

#### SpringBoot允许程序在命令行参数中指定 定义在properties文件中的参数值并覆盖

比如上面我们配置了server.port作为运行时tomcat端口, 但是假如我们使用如下命令编译除了jar包, 在使用java -jar 运行应用的时候, 我们可以指定--server.port=8080来覆盖配置文件中的配置, 并且在8080端口上面启动tomcat

下面我们就来尝试一下, 运行:
	
	mvn clean install -Dmaven.test.skip

打包出来jar包, 我们切换到jar包所在的目录下面, 使用如下命令查看命令行输出:

	java -jar springboot-0.0.1-SNAPSHOT.jar --server.port=8080

我们可以看到输出中已经运行的端口为8080端口

![命令行参数改变配置文件参数配置](/assets/images/2017-09-11-spring-command-param.png)

### Spring配置文件属性映射到Bean上面

#### 首先我们创建一个book的properties文件, 添加如下信息

	book.author=terrylmay
	book.isbn=201982392
	book.publishDate=2017-09-11

#### 创建与properties对应的Bean

	package com.terrylmay.springboot;

	import org.springframework.boot.context.properties.ConfigurationProperties;
	import org.springframework.context.annotation.PropertySource;
	import org.springframework.stereotype.Component;

	@Component
	@ConfigurationProperties(prefix = "book")
	@PropertySource(value = { "classpath:book.properties" })
	public class BookSetting {
		private String author;
		private String isbn;
		private String publishDate;

		public String getAuthor() {
			return author;
		}

		public void setAuthor(String author) {
			this.author = author;
		}

		public String getIsbn() {
			return isbn;
		}

		public void setIsbn(String isbn) {
			this.isbn = isbn;
		}

		public String getPublishDate() {
			return publishDate;
		}

		public void setPublishDate(String publishDate) {
			this.publishDate = publishDate;
		}

	}

#### 开发restful API测试效果

	package com.terrylmay.springboot;

	import org.springframework.beans.factory.annotation.Autowired;
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

		@Autowired
		BookSetting bookSetting;

		@RequestMapping(value = "/sayHello", method = RequestMethod.GET)
		public String sayHello() {
			System.out.println(bookSetting.getIsbn());
			return bookSetting.getAuthor();
		}
	}

#### 开发测试用例, 测试效果

	package com.terrylmay.springboot;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	import org.springframework.test.context.web.WebAppConfiguration;

	/**
	 * Unit test for simple App.
	 */
	@RunWith(SpringJUnit4ClassRunner.class)
	@SpringBootTest(classes = App.class)
	@WebAppConfiguration
	public class AppTest {

		@Autowired
		BookSetting bookSetting;

		@Test
		public void should_return_author() {
			assertTrue(bookSetting.getAuthor().equals("terrylmay"));
		}
	}

上面就是SpringBoot里面的基本配置的使用.





