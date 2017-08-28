---
layout: post
title: Spring中条件注解的用法
date: 2017-08-28 23:37:37
tags: [Spring, 注解Condition]
---

前面有一篇记录Profile注解的使用, 确保根据传入的不同的参数获取不同的数据库访问的Bean, 使用Profile能完成的事情Condition都能完成,并且Condition能区分更多的不同条件下Bean的创建。比如当A Bean创建之后才会创建B Bean; 比如在Linux 列出目录下所有文件 需要创建一个WindowsBean 而linux下列出所有的目录需要创建一个LinuxBean, 但他们的接口都是IOSCommand.

## 使用Condition模拟一下Profile

#### 创建数据库资源文件接口

	package com.terrylmay;

	public interface IDBResourceConfig {
		public String getDBURL();

		public String getDBUserName();

		public String getDBPassword();
	}

#### 创建Dev环境下条件注解

	package com.terrylmay;

	import org.springframework.context.annotation.Condition;
	import org.springframework.context.annotation.ConditionContext;
	import org.springframework.core.type.AnnotatedTypeMetadata;

	public class DevDBCondition implements Condition {

		public boolean matches(ConditionContext context, AnnotatedTypeMetadata arg1) {
			String[] profiles = context.getEnvironment().getActiveProfiles();
			boolean match = false;
			for (String profile : profiles) {
				if (profile.equals("dev")) {
					match = true;
					break;
				}
			}
			return match;
		}

	}

#### 创建生产环境下的条件注解

	package com.terrylmay;

	import org.springframework.context.annotation.Condition;
	import org.springframework.context.annotation.ConditionContext;
	import org.springframework.core.type.AnnotatedTypeMetadata;

	public class ProdDBCondition implements Condition {

		public boolean matches(ConditionContext context, AnnotatedTypeMetadata arg1) {
			String[] profiles = context.getEnvironment().getActiveProfiles();
			boolean match = false;
			for (String profile : profiles) {
				if (profile.equals("prod")) {
					match = true;
					break;
				}
			}
			return match;
		}

	}

#### 创建Dev环境下的资源访问Bean

	package com.terrylmay;

	import org.springframework.context.annotation.Conditional;
	import org.springframework.stereotype.Component;

	@Component
	@Conditional(DevDBCondition.class)
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

#### 创建Prod环境下的资源访问Bean

	package com.terrylmay;

	import org.springframework.context.annotation.Conditional;
	import org.springframework.stereotype.Component;

	@Component
	@Conditional(ProdDBCondition.class)
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

#### 创建配置文件

	package com.terrylmay;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class BeanConfig {

	}

#### 创建启动类

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
			context.getEnvironment().setActiveProfiles("dev");
			context.register(BeanConfig.class);
			context.refresh();

			IDBResourceConfig dbResource = context.getBean(IDBResourceConfig.class);
			System.out.println(dbResource.getDBURL());
			context.close();
		}
	}

上述步骤, 我们使用Condition注解来模拟了Profile的使用。





	