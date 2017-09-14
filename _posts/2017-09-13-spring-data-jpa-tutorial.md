---
layout: post
title: SpringBoot中数据访问JPA的使用
date: 2017-09-13 21:30:00
tags: [SpringBoot, JPA]
---

我们基于上篇用到的springboot工程来完成SpringBoot Jpa的基本使用实验。SpringData JPA 是Spring Data的一个子项目, 用于简化对于数据库操作的代码。

在练习使用SpringDataJPA之前, 我们需要先安装一下mysql数据库。具体的数据库安装教程在这里就不多说了。

## 创建数据库 以及 User数据表

	create database spring_data_jpa_test;
	use spring_data_jpa_test;
	create table user(id int primary key not null auto_increment, username varchar(32) unique not null, password varchar(32) not null, email varchar(32) unique not null);

## 添加Spring Data Jpa依赖并且 添加mysql驱动依赖

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-jpa</artifactId>
		<version>1.5.6.RELEASE</version>
	</dependency>

	<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>6.0.6</version>
	</dependency>

## 配置JPA 的参数 以及 数据库连接池Bean

### 在src/main/resources/下创建application.properties, 并配置spring jpa参数

	spring.datasource.driverClassName = com.mysql.cj.jdbc.Driver
	spring.datasource.url = jdbc:mysql://127.0.0.1:3306/spring_data_jpa_test
	spring.datasource.username = root
	spring.datasource.password = welcome

	spring.jpa.hibernate.ddl-auto = update
	spring.jpa.show-sql = true
	spring.jackson.serialization.indent_output=true

上面的代码中用来配置数据源 以及 jpa,分别以spring.datasource 以及 spring.jpa为前缀。

### 创建与表对应的实体类

	package com.terrylmay.springboot.entity;

	import javax.persistence.Column;
	import javax.persistence.Entity;
	import javax.persistence.GeneratedValue;
	import javax.persistence.Id;

	@Entity(name = "user")
	public class UserEntity {

		@Id
		@GeneratedValue
		private Integer id;

		@Column(name = "username", nullable = false, unique = true, updatable = true, length = 32)
		private String username;

		@Column(name = "password", nullable = false, length = 32)
		private String password;

		@Column(name = "email", nullable = false, unique = true, length = 32)
		private String email;

		public UserEntity(String username, String password, String email) {
			super();
			this.username = username;
			this.password = password;
			this.email = email;
		}
		//此处省略get 和 set方法
	}

在上面代码中, 对于每一个字段都进行了注解; 这样的话代码里面的变量名 与 数据库里面的变量对应关系一目了然. 其实 字段也是可以不用注解的, 如果使用@Entity对类进行了注解, 只要类中的成员变量符合相关的规则, 同样也是可以操作数据库的. 比如: 如果数据库表中有一个字段叫book_author, 那么代码中的变量只需要指定为bookAuthor即可完成对字段的操作, 即Java代码中的驼峰命名法对应数据库字段中的匈牙利命名。如果字段都是小写 比如 age 那么代码中也只需要写成age即可。但是有个缺点就是对于不了解这个规则的人来说会比较难读一些。

### 创建对应的数据库访问层Repository

	package com.terrylmay.springboot.repository;

	import org.springframework.data.jpa.repository.JpaRepository;

	import com.terrylmay.springboot.entity.UserEntity;

	public interface UserRepository extends JpaRepository<UserEntity, Integer> {
		public UserEntity findByUsername(String username);

		public UserEntity findByUsernameAndEmail(String username, String email);

	}

我们只需要定义接口, 并且继承JpaRepository, 然后根据相应的代码规则写出代码即可. 比如我想通过username来查找相应的UserEntity就可以直接写成上面这种代码, 并不需要实现即可完成查数据的功能

### 定义RestController来验证数据库访问层功能

为简单起见, 我们直接在Application启动类中加入访问方法

	
	package com.terrylmay.springboot;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.SpringApplication;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.web.bind.annotation.PathVariable;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;
	import org.springframework.web.bind.annotation.RestController;

	import com.terrylmay.springboot.entity.UserEntity;
	import com.terrylmay.springboot.repository.UserRepository;

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

		@Autowired
		UserRepository userRepository;

		@RequestMapping(value = "/sayHello", method = RequestMethod.GET)
		public String sayHello() {
			System.out.println(bookSetting.getIsbn());
			return bookSetting.getAuthor();
		}

		@RequestMapping(value = "/getUserByName/{username}", method = RequestMethod.GET)
		public String getUserByName(@PathVariable(value = "username") String userName) {
			return userRepository.findByUsername(userName).getUsername();
		}

		@RequestMapping(value = "/registe/user/{username}/{password}/{email}")
		public String registeUser(@PathVariable(value = "username") String userName,
				@PathVariable(value = "password") String password, @PathVariable(name = "email") String email) {
			UserEntity userEntity = new UserEntity(userName, password, email);
			userEntity = userRepository.save(userEntity);

			return userEntity.getId() + "";
		}
	}

开发注册接口 以及 查找接口, 下面我们启动应用来看一下功能是否可用。

### 启动应用程序验证功能

启动应用程序, 我们可以看到日志输出, 已经加入了JPA的功能

	Wed Sep 13 22:34:05 CST 2017 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
	Wed Sep 13 22:34:05 CST 2017 WARN: Establishing SSL connection without server's identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
	2017-09-13 22:34:05.652  INFO 8921 --- [           main] j.LocalContainerEntityManagerFactoryBean : Building JPA container EntityManagerFactory for persistence unit 'default'
	2017-09-13 22:34:05.688  INFO 8921 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [
		name: default
		...]
	2017-09-13 22:34:05.849  INFO 8921 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.0.12.Final}
	2017-09-13 22:34:05.851  INFO 8921 --- [           main] org.hibernate.cfg.Environment            : HHH000206: hibernate.properties not found
	2017-09-13 22:34:05.853  INFO 8921 --- [           main] org.hibernate.cfg.Environment            : HHH000021: Bytecode provider name : javassist
	2017-09-13 22:34:05.926  INFO 8921 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.0.1.Final}
	2017-09-13 22:34:06.143  INFO 8921 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.MySQL5Dialect
	2017-09-13 22:34:06.989  INFO 8921 --- [           main] org.hibernate.tool.hbm2ddl.SchemaUpdate  : HHH000228: Running hbm2ddl schema update
	2017-09-13 22:34:07.120  INFO 8921 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
	
这时候我们发起网络请求

	http://localhost:9080/springboot/registe/user/terrylmay/Pr0d1234/1575639478@qq.com

因为application.properties中设置了上下文根, 所以访问路径中需要加上上下文根才能正确访问

我们可以看到, 数据中有数字1返回, 因为我们只返回了userEntity的ID 且ID是自增的, 所以插入第一条数据的Id为1
![插入数据](/assets/images/2017-09-13-spring-data-jpa-save-user.png)

这是我们可以看到我们的应用控制台同时打印出了hql语句, 这是因为我们配置了spring.jpa.show-sql=true这个配置

![控制台插入数据](/assets/images/2017-09-13-spring-data-jpa-save-console.png)

我们再去Mysql workbench中查看user表中的数据, 可以查到刚才插入的数据

![mysql数据查询](/assets/images/2017-09-13-spring-data-jpa-mysql.png)

随后我们发起查询的网络请求, 应该可以获取到刚才插入的用户名信息

![查询返回用户名](/assets/images/2017-09-13-spring-data-jpa-getbyname.png)

同时, 我们在应用的管理控制台也同样可以看到相关查询的hql语句

![查询HQL语句](/assets/images/2017-09-13-spring-data-jpa-get-console.png)

到这里 使用spring data jpa 就算是初步完成了, 后面再去看如何配置数据库连接池以及其他jpa的高级用法。


	