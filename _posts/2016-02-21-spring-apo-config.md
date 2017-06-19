---
layout: post
title: Spring中AOP配置
date: 2016-02-21 10:51:34
tags: [Spring, AOP]
---

Spring中AOP配置的两种方式包括XML文件配置 以及 使用Annotation方式配置

xml配置

	<?xml version="1.0" encoding="UTF-8"?>  
	<beans xmlns="http://www.springframework.org/schema/beans"  
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"  
	    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"  
	    xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"  
	    xsi:schemaLocation="  
	            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
	            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
	            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd  
	            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd  
	            ">  
	    <context:component-scan base-package="com.maylor"></context:component-scan>  
	    <bean id="logIntereptor" class="com.maylor.system.aop.LogInterceptor"></bean>  
	    <!-- 配置aop切面以及切入点 -->  
	    <aop:config>  
	    <!-- 定义pointcut -->  
	        <aop:pointcut  
	            expression="execution (public void com.maylor.system.dao.impl.UserDAOImpl.addUser(..))"  
	            id="servicesPointCut" />  
	            <!-- 定义切面类,并且为其配置method应该在那个切入点执行 -->  
	        <aop:aspect id="logAspect" ref="logIntereptor">  
	            <aop:before method="before" pointcut-ref="servicesPointCut" />  
	        </aop:aspect>  
	    </aop:config>  
	</beans>  

代码配置：

	package com.maylor.system.aop;

	import org.aspectj.lang.annotation.Aspect;  
	import org.aspectj.lang.annotation.Before;  
	import org.springframework.stereotype.Component;  


	@Aspect  
	@Component  
	public class LogInterceptor {  
	    @Before("execution (public void com.maylor.system.dao.impl.UserDAOImpl.addUser(..))")  
	    public void before() {  
	    System.out.println("before start");  
	    }  
	}
