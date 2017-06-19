---
layout: post
title: Websocket入门以及工程搭建
date: 2016-02-21 10:39:43
tags: [Websocket]
---

websocket伴随Html5兴起这么长时间以来,我一直都想探一下websocket的奥秘.可是总是没有时间来看相关资料..今天感觉没什么事.而且有点闲,查看了一些别人的博客.基本了解了websocket在代码层的实现了.以后要做的就是关于websocket的原理了..首先动手写一个关于websocket的聊天程序吧.当然 这个聊天程序也是基于j2ee的..客户端用原生的js来实现.首先要想使用websocket ，那么jdk的版本一定要是1.7以上的..websocket是1.7之后加的新特性.

<!-- more -->

那么我们开始代码层面的开发吧..
首先 建立服务器端程序..

	package com.system.maylor.test;  

	import java.io.IOException;  
	import java.nio.ByteBuffer;  
	import java.nio.CharBuffer;  
	import java.util.ArrayList;  

	import javax.servlet.http.HttpServletRequest;  

	import org.apache.catalina.websocket.MessageInbound;  
	import org.apache.catalina.websocket.StreamInbound;  
	import org.apache.catalina.websocket.WebSocketServlet;  
	import org.apache.catalina.websocket.WsOutbound;  

	public class WebSocketTest extends WebSocketServlet {  

	    private static final long serialVersionUID = -4853540828121130946L;  
	    private static ArrayList<MyMessageInbound> mmiList = new ArrayList<MyMessageInbound>();  

	    @Override  
	    protected StreamInbound createWebSocketInbound(String str,  
	            HttpServletRequest request) {  
	        return new MyMessageInbound();  
	    }  

	    private class MyMessageInbound extends MessageInbound {  
	        WsOutbound myoutbound;  

	        @Override  
	        public void onOpen(WsOutbound outbound) {  
	            try {  
	                System.out.println("Open Client.");  
	                this.myoutbound = outbound;  
	                mmiList.add(this);  
	                outbound.writeTextMessage(CharBuffer.wrap("Hello!"));  
	            } catch (IOException e) {  
	                e.printStackTrace();  
	            }  
	        }  

	        @Override  
	        public void onClose(int status) {  
	            System.out.println("Close Client.");  
	            mmiList.remove(this);  
	        }  
	                //当服务器端收到信息的时候,就对所有的连接进行遍历并且把收到的信息发送给所有用户  
	        @Override  
	        public void onTextMessage(CharBuffer cb) throws IOException {  
	            System.out.println("Accept Message : " + cb);  
	            for (MyMessageInbound mmib : mmiList) {  
	                CharBuffer buffer = CharBuffer.wrap(cb);  
	                mmib.myoutbound.writeTextMessage(buffer);  
	                mmib.myoutbound.flush();  
	            }  
	        }  

	        @Override  
	        public void onBinaryMessage(ByteBuffer bb) throws IOException {  
	        }  
	    }  

	}

在服务器端创建一个servlet,继承自WebSocketServlet，当然也可以使用注解的方式实现..在这就不写注解的代码了.servlet创建好之后 当然要对servlet进行配置了.在web.l文件中进行配置

	<display-name>Websocket</display-name>  
	<servlet>  
	    <servlet-name>wsServlet</servlet-name>  
	    <servlet-class>com.system.maylor.test.WebSocketTest</servlet-class>  
	</servlet>  
	<servlet-mapping>  
	    <servlet-name>wsServlet</servlet-name>  
	    <url-pattern>/wsServlet</url-pattern>  
	</servlet-mapping>  

把上面一段配置信息加入到你的web.xml文件中..这样服务器端的websocket就建立成功了..

下面就是写客户端的代码了.index.html

	<!DOCTYPE html>  
	<html>  
		<head>  
			<meta charset=UTF-8>  
			<title>Tomcat WebSocket Chat</title>  
			<script>  
			[html] view plaincopyprint?在CODE上查看代码片派生到我的代码片
			//建立客户端websocket  
			    var ws = new WebSocket("ws://localhost:8080/Websocket/wsServlet");  
			[html] view plaincopyprint?在CODE上查看代码片派生到我的代码片
			       //定义ws的一些回调函数  
			ws.onopen = function() {  
			};  
			[html] view plaincopyprint?在CODE上查看代码片派生到我的代码片
			//当有消息传过来时执行的操作  
			    ws.onmessage = function(message) {  
			        document.getElementById("chatlog").textContent += message.data + "\n";  
			    };  
			    function postToServer() {  
			        ws.send(document.getElementById("msg").value);  
			        document.getElementById("msg").value = "";  
			    }  
			    function closeConnect() {  
			        ws.close();  
			    }  
			</script>  
		</head>  
		<body>  
		    <textarea id="chatlog" readonly></textarea>  
		    <br />  
		    <input id="msg" type="text" />  
		    <button type="submit" id="sendButton" onClick="postToServer()">Send!</button>  
		    <button type="submit" id="sendButton" onClick="closeConnect()">End</button>  
		</body>  
	</html>  

这样基本上一个大厅聊天的程序就已经完成了.启动服务器.然后打开多个浏览器.同时访问index.

html界面当某个浏览器发出消息时，在其他浏览器或者其他界面就可以收到这个消息了..因为这个是一个比较简单的聊天程序.

不过想想最近做的一个基于socket的聊天程序.这样的代码确实简洁了不少.如果用socket编程的话,至少要开启两个线程.一个线程用来监听连接请求..另一个线程则用来监听处理某一个socket发来消息时，服务器的转发给所有的socket..我想可能websocket也是通过这种方式实现的.

但至少在代码简洁层面上让我们更加容易读懂代码了一些,还有就是我感觉websocket比较智能的一点就是当连接断开时,服务器会知道这二个连接已断开.并且进行相应的处理.而socket不会监听到客户端socket断开的请求..必须在客户端手动的发送某一个特殊字符,比如end然后服务器端当接到这个end字符的时候认定客户端socket已经断开..以上是我关于socket的体会吧..有不对的地方欢迎指出.批评.
