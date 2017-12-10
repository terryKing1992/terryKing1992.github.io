---
layout: post
title: Spring中测试框架的使用
date: 2017-08-29 22:00:00
tags: [Spring, 单元测试]
---

单元测试与集成测试是保证程序可靠运行的基础。据可靠统计如果你单元测试的代码量是本身程序功能代码量的5倍的时候, 系统出Bug的概率会大大的减少。所以单元测试与集成测试是开发程序不可缺少的一部分, 同时单元测试与集成测试的代码编写也是程序员需要掌握的。

#### 首先在工程中加入依赖

使用maven管理工程依赖, 在maven工程依赖管理文件pom.xml中添加如下内容:

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
		<modelVersion>4.0.0</modelVersion>

		<groupId>com.terrylmay</groupId>
		<artifactId>com.terrylmay</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<packaging>jar</packaging>

		<name>com.terrylmay</name>
		<url>http://maven.apache.org</url>

		<properties>
			<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		</properties>

		<dependencies>
			<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
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
			<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
			<dependency>
				<groupId>org.springframework</groupId>
				<artifactId>spring-test</artifactId>
				<version>4.3.9.RELEASE</version>
				<scope>test</scope>
			</dependency>

			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>4.12</version>
				<scope>test</scope>
			</dependency>
		</dependencies>
	</project>

#### 添加应用功能模块

1、创建一个模拟数据库访问类UserRepository, 这里也就不为UserRepository抽象接口了

	package com.terrylmay;

	import org.springframework.stereotype.Component;

	@Component
	public class UserRepository {
		public String getUserNameById(String userId) {
			return "terry" + userId;
		}
	}

2、创建一个服务类, 用于给上层提供服务

	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Service;

	@Service
	public class UserService {
		@Autowired
		UserRepository userRepository;

		public String getUserNameById(String userId) {
			return userRepository.getUserNameById(userId);
		}
	}

3、创建测试类, 完成对UserRepository的测试

	package com.terrylmay;

	import static org.junit.Assert.*;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = { ApplicationTestConfig.class })
	public class UserRepositoryTest {

		@Autowired
		UserRepository userRepository;

		@Test
		public void getUserNameById_should_return_terry_userId() {
			String userName = userRepository.getUserNameById("123");
			assertEquals(userName, "terry123");
		}
	}

4、 创建测试类, 完成对UserService的测试
	
	package com.terrylmay;

	import static org.junit.Assert.*;

	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = { ApplicationTestConfig.class })
	public class UserServiceTest {
		@Autowired
		UserService userService;

		@Test
		public void getUserNameById_should_return_terry_userId() {
			String userName = userService.getUserNameById("123");
			assertEquals(userName, "terry123");
		}
	}

可以使用eclipse或者Intellij中的Run按钮开启一个一个类测试

在该测试类中之所以会使用ContextConfiguration注解, 是因为本类中没有使用到application.properties等外部加载文件, 
只需要加载SpringContext环境, 并且完成对Bean的自动注入即可. 在实际的生产实践中, 需要使用SpringContextConfiguration注解来对Test类进行注解, 因为SpringContextConfiguration可以完全模拟SpringBootApplication的应用启动过程。

5、 使用Maven运行单元测试

使用命令运行单元测试

	mvn clean install -Dmaven.test.skip=false

如果能看到如下结果则说明单元测试是已经通过了的
![测试结果](/assets/images/2017-08-29-spring_run_test.png)


