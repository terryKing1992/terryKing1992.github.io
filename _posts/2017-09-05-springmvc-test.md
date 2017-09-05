---
layout: post
title: SpringMVC如何做集成测试
date: 2017-09-05 23:00:00
tags: [Spring, MVC, 测试]
---

测试是保证软件成功的关键.如果在SpringMVC中想要不启动程序就测试程序中开发的服务, 我们就需要使用Servlet的模拟对象,比如:MockMVC、MockHttpServletRequest、MockHttpServletResponse、MockHttpSessiion等来帮我们完成集成测试。在下面的示例中我们将演示SpringMVC Test测试框架的使用. 当然还是基于前面已经搭建好的SpringMVC工程进行试验。

### 添加Maven依赖

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
		<version>3.8.1</version>
		<scope>test</scope>
	</dependency>

### 添加SpringMVC配置类

	package com.terrylmay;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.config.annotation.EnableWebMvc;

	@Configuration
	@EnableWebMvc
	@ComponentScan("com.terrylmay")
	public class SpringMVCTestConfig {

	}

### 开发HelloWorldController类

	package com.terrylmay;

	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;

	@RestController
	public class HelloController {

		private static final Logger LOG = LoggerFactory.getLogger(HelloController.class);

		@RequestMapping(value = "/hello", produces = "text/plain;charset=UTF-8")
		public String sayHello() {
			LOG.info("Hello 接口被调用了");
			return "Hello World";
		}
	}


### 开发Test测试类

	package com.terrylmay;

	import org.junit.Before;
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	import org.springframework.test.context.web.WebAppConfiguration;
	import org.springframework.test.web.servlet.MockMvc;
	import org.springframework.test.web.servlet.setup.MockMvcBuilders;
	import org.springframework.web.context.WebApplicationContext;

	import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
	import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = { SpringMVCTestConfig.class })
	@WebAppConfiguration
	public class HelloControllerTest {

		MockMvc mockMvc;

		@Autowired
		WebApplicationContext webApplicationContext;

		@Before
		public void setup() {
			this.mockMvc = MockMvcBuilders.webAppContextSetup(this.webApplicationContext).build();
		}

		@Test
		public void testSayHello() throws Exception {
			this.mockMvc.perform(get("/hello")).andExpect(status().isOk()).andExpect(content().string("Hello World"))
					.andExpect(content().contentType("text/plain;charset=UTF-8"));
		}
	}

运行测试用例, 可以看到运行结果

![测试用例运行成功](/assets/images/2017-09-05-spring-mvc-test-success.png)


