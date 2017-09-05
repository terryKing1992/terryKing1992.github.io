---
layout: post
title: SpringMVC 中常用的注解
date: 2017-09-01 22:00:00
tags: [Spring, MVC, 注解]
---

SpringMVC中的注解包括@Controller、@RestController、@ResponseBody、@RequestMapping、@PathVariable、@RequestBody等等, 这些注解在一个或者多个Controller类中使用, 能够完成各种Http请求 以及 不同参数的处理, 同时能够进行参数的各种个性化定义

下面我们就来看一下Controller中常见的注解都有什么作用

## @Controller注解

@Controller注解在类上, 表明这个类是一个Controller类, 负责处理用户的http请求。并将该类声明为一个Spring Bean, Dispatcher Servlet会自动扫描注解了此注解的类, 并将Web请求映射到该类的@RequestMapping方法指定的路径上面去。 注意如果是仅仅声明一个普通的Bean, 那么@Service、@Component、@Repository、@Controller的作用是一模一样的, 可以相互替换, 如果是声明一个用于处理用户Http请求的类时, 只能使用@Controller来声明Bean

## @RequestMapping注解

该注解是用来映射Web请求(访问路径与访问参数), @RequestMapping 可以注解在类上面也可以注解在方法上, 注解在方法上面的路径会继承类上面的访问路径, 例如 @RequestMapping("/entity") 注解在类上面, @RequestMapping("/create")注解在该类的某一个方法上面, 那么如果访问方法对应的Restful服务的话, 访问路径为hostname:port/上下文根/entity/create。@RequestMapping 支持Servlet的request 和 response作为参数

## @ResponseBody

@ResponseBody 支持将返回值放到response中, 而不是返回一个界面。可以返回JSON类型的字符串, 也可以返回Map等数据。

## PathVariable

用于接收路径参数 如/entity/user

## @RestController

@RestController是一个组合注解, 组合了@Controller 和 @ResponseBody, 这意味着当开发一个Restful的接口的时候可以使用此注解

## 注解示例

### 添加Spring MVC 依赖以及jackson的依赖

在上一篇博客的基础上, 添加jackson的依赖

	<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
	<dependency>
	    <groupId>com.fasterxml.jackson.core</groupId>
	    <artifactId>jackson-databind</artifactId>
	    <version>2.9.0</version>
	</dependency>

刷新工程依赖之后, 我们会发现依赖的jar包列表中增加了3个jackson的包

![Jackson包依赖](/assets/images/2017-09-05-spring-jackson-dependency.png)

### 创建一个User对象, 用于演示rest接口返回对象示例

	package com.terrylmay.model;

	public class User {
		private String userName;
		private Integer age;

		public User() {

		}

		public User(String userName, Integer age) {
			super();
			this.userName = userName;
			this.age = age;
		}

		public String getUserName() {
			return userName;
		}

		public void setUserName(String userName) {
			this.userName = userName;
		}

		public Integer getAge() {
			return age;
		}

		public void setAge(Integer age) {
			this.age = age;
		}

	}

### 创建Controller对象, 来演示Controller中注解的使用

	
#### 方法中使用HttpServletRequest

	package com.terrylmay.controller;

	import javax.servlet.http.HttpServletRequest;

	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	@RequestMapping("/root")
	public class AnnotationUsingController {

		@RequestMapping(value = "/useHttpServletRequest", produces = "text/plain; charset=utf-8")
		public String useHttpRequestInMethod(HttpServletRequest request) {
			return request.getRequestURL() + "access success";
		}
	}

在上例中我们演示了@RequestMapping注解在类上面与注解在方法上面, 访问的时候需要使用/root/useHttpServletRequest路径来访问, 同时以HttpServletRequest作为参数 可以接收到Request中携带的其他的一些参数。

我们通过浏览器访问该Get方法, 可以看到下图中网络请求的结果

![用户网络请求截图](2017-09-05-spring-using-httpservlet-request.png)

#### 访问接口中使用单路径参数

	package com.terrylmay.controller;

	import javax.servlet.http.HttpServletRequest;

	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	@RequestMapping("/root")
	public class AnnotationUsingController {

		@RequestMapping(value = "/useHttpServletRequest", produces = "text/plain; charset=utf-8")
		public String useHttpRequestInMethod(HttpServletRequest request) {
			return request.getRequestURL() + "access success";
		}

		@RequestMapping(value = "/path/{userName}", produces = "text/plain; charset=utf-8")
		public String useSinglePathVariable(@PathVariable String userName, HttpServletRequest request) {
			return "当前路径参数中userName=" + userName + ";当前访问URL为:" + request.getRequestURI();
		}
	}

当我们访问如下路径时:
	
	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/path/terrylmay

我们会得到如下结果

	当前路径参数中userName=terrylmay;当前访问URL为:/com.terrylmay-0.0.1-SNAPSHOT/root/path/terrylmay

说明我们从路径参数{userName}中获取到了路径中的参数


#### 访问接口中使用多路径参数

	
	package com.terrylmay.controller;

	import javax.servlet.http.HttpServletRequest;

	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	@RequestMapping("/root")
	public class AnnotationUsingController {

		@RequestMapping(value = "/useHttpServletRequest", produces = "text/plain; charset=utf-8")
		public String useHttpRequestInMethod(HttpServletRequest request) {
			return request.getRequestURL() + "access success";
		}

		@RequestMapping(value = "/path/{userName}", produces = "text/plain; charset=utf-8")
		public String useSinglePathVariable(@PathVariable String userName, HttpServletRequest request) {
			return "当前路径参数中userName= " + userName + ";当前访问URL为:" + request.getRequestURI();
		}

		@RequestMapping(value = "/path/{userName}/{age}", produces = "text/plain; charset=utf-8")
		public String useMultiPathVariable(@PathVariable(value = "userName") String userName,
				@PathVariable(value = "age") Integer age, HttpServletRequest request) {
			return "当前路径参数中userName=" + userName + ",当前路径中age=" + age + ",当前URL为:" + request.getRequestURI();
		}
	}

当我们访问如下路径时:
	
	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/path/terrylmay/19

我们会得到如下结果

	当前路径参数中userName=terrylmay,当前路径中age=19,当前URL为:/com.terrylmay-0.0.1-SNAPSHOT/root/path/terrylmay/19

单路径参数与多路径参数在使用上的唯一一点不同就是多路径参数的@PathVariable中需要指定该变量代表的是哪一个参数, 比如@PathVariable(value="userName") 就表示路径中标识为{userName}的这个位置的变量

当然如果按照路径参数的顺序写方法中的变量也是可以的, 比如如下代码:

	@RequestMapping(value = "/path/{userName}/{age}", produces = "text/plain; charset=utf-8")
	public String useMultiPathVariable(@PathVariable String userName, @PathVariable Integer age,
			HttpServletRequest request) {
		return "当前路径参数中userName=" + userName + ",当前路径中age=" + age + ",当前URL为:" + request.getRequestURI();
	}


#### Get请求在?后面加上访问参数

	@RequestMapping(value = "/getUserAction.do", produces = "text/plain; charset=utf-8")
	public String useGetParameter(String userName, HttpServletRequest request) {
		return "当前路径参数中userName=" + userName + ",当前URL为:" + request.getRequestURI();
	}

当方法参数中有一个没有任何注解的变量存在时, 那么访问的时候就需要使用?来传递参数

	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/getUserAction.do?userName=terrylmay

这样我们就可以得到返回值

	当前路径参数中userName=terrylmay,当前URL为:/com.terrylmay-0.0.1-SNAPSHOT/root/getUserAction.do

#### 接收JSON格式的对象参数

	@RequestMapping(value = "/user", produces = "application/json; charset=utf-8")
	public String getUser(User user, HttpServletRequest request) {
		return "当前得到的Json对象为, userName:" + user.getUserName() + ",age:" + user.getAge();
	}

当访问该方法的时候, 对应的访问路径为:
	
	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/user?userName=terrylmay&age=20

得到的结果为:

	当前得到的Json对象为, userName:terrylmay,age:20


#### 多访问路径的写法

	@RequestMapping(value = { "/user", "user1" }, produces = "application/json; charset=utf-8")
	public String getUserClone(User user, HttpServletRequest request) {
		return "当前得到的Json对象为, userName" + user.getUserName() + ",age:" + user.getAge();
	}

对应的访问路径为:

	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/user1?userName=terrylmay&age=20

	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/user2?userName=terrylmay&age=20

得到的访问结果都是一样的:

	当前得到的Json对象为, userNameterrylmay,age:20

#### 直接返回JSON对象

	@RequestMapping(value = { "/getUserJsonData" }, produces = "application/json; charset=utf-8")
	public User getUserJsonObject(HttpServletRequest request) {
		return new User("terrylmay", 21);
	}

当我们访问如下路径的时候:

	http://localhost:8080/com.terrylmay-0.0.1-SNAPSHOT/root/getUserJsonData

我们会得到JSON类型的数据, 如下:

	{"userName":"terrylmay","age":21}










