---
layout: post
title: Spring中Event的使用
date: 2017-08-26 23:37:37
tags: [Spring, Event]
---

事件机制属于异步机制, 基本上在所有的代码框架中都会存在事件驱动模块供开发者调用; 用在当某一个事情做完了之后, 需要触发另外一系列的动作的时候使用.比如微信收到别人发送过来的一条消息之后, 首先微信的聊天列表会显示小红点信息, 同时需要更新数据库记录, 如果用户刚好在聊天界面, 聊天界面的对话列表也需要更新, 这时候就非常适合使用Event事件驱动来完成这项工作, 定义多个Listener来监听同一个事件 并且完成不同的工作。

同样, Spring也提供了相同的机制, 下面就看一下Spring 事件机制是如何使用的

## 创建一个单聊消息事件需要继承ApplicationEvent类,

	package com.terrylmay;

	import org.springframework.context.ApplicationEvent;

	public class SingleChatMessageEvent extends ApplicationEvent {

		/**
		 * 
		 */
		private static final long serialVersionUID = -3717121192162377719L;

		private String messageFrom;

		private String messageTo;

		private String messageContent;

		public SingleChatMessageEvent(Object source, String messageFrom, String messageTo, String messageContent) {
			super(source);
			this.messageFrom = messageFrom;
			this.messageTo = messageTo;
			this.messageContent = messageContent;
		}

		public String getMessageFrom() {
			return messageFrom;
		}

		public void setMessageFrom(String messageFrom) {
			this.messageFrom = messageFrom;
		}

		public String getMessageTo() {
			return messageTo;
		}

		public void setMessageTo(String messageTo) {
			this.messageTo = messageTo;
		}

		public String getMessageContent() {
			return messageContent;
		}

		public void setMessageContent(String messageContent) {
			this.messageContent = messageContent;
		}
	}

## 创建单聊消息监听类

	package com.terrylmay;

	import org.springframework.context.ApplicationListener;
	import org.springframework.stereotype.Component;

	@Component
	public class SingleChatMessageListener implements ApplicationListener<SingleChatMessageEvent> {

		public void onApplicationEvent(SingleChatMessageEvent chatMessage) {
			System.out.println(
					String.format("收到%s消息, 消息内容为%s", chatMessage.getMessageFrom(), chatMessage.getMessageContent()));
		}

	}

## 同时, 我们需要定义聊天消息的发布类

假设我们现在从Socket中获取到了数据流, 并且使用相应的Decoder进行了数据的解析, 随后我们需要调用一个事件的发射器类似的东西来将消息丢入到Listener监听的队列中去

	package com.terrylmay;

	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.ApplicationContext;
	import org.springframework.stereotype.Component;

	@Component
	public class SingleChatMessagePublisher {
		@Autowired
		ApplicationContext applicationContext;

		public void publishSingleChatMessage(SingleChatMessageEvent singleChatMessageEvent) {
			applicationContext.publishEvent(singleChatMessageEvent);
		}
	}

## 系统的配置类还是需要创建的

	package com.terrylmay;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;

	@Configuration
	@ComponentScan("com.terrylmay")
	public class BeanConfig {

	}

## 定义应用入口程序来验证事件机制

	package com.terrylmay;

	import java.io.IOException;

	import org.springframework.context.annotation.AnnotationConfigApplicationContext;

	public class ApplicationEntry {
		public static void main(String[] args) throws IOException {
			AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfig.class);
			SingleChatMessagePublisher publisher = context.getBean(SingleChatMessagePublisher.class);
			publisher.publishSingleChatMessage(new SingleChatMessageEvent(publisher, "terry", "may", "Yes, you are right"));
			context.close();
		}
	}

运行程序之后, 我们可以看到程序控制台输出了Listener中定义的打印:

	收到terry消息, 消息内容为Yes, you are right

Spring中的事件机制就是这样




