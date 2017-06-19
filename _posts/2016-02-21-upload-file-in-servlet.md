---
layout: post
title: Servlet文件上传实现
date: 2016-02-21 10:32:53
tags: [Servlet]
---

由于最近看到有人论坛上发帖称自己用了网上的某个框架,因为是在裸机上面开发的,并没有检测到病毒,等用其他机器演示的时候整个项目出现了病毒，并且客户直接否认了这个项目，突然让我想到了学长说的一句话：工具类我一般自己写，谁知道他们写成什么样子给你用呢。又鉴于文件上传功能比较常见，而且大部分人都还在使用像SmartUpload等组件.而我实在是不想使用这类工具了，所以打算自己动手写一个文件上传的实现

<!-- more -->

2.实现思路

首先，通过分析表单提交的内容到底包含一些什么内容,然后逐步解析，从而得到自己想要的文件信息,最后通过流的形式写入服务器硬盘中.

3.功能实现

首先，使用IDE自动生成一个web工程，在web工程中创建servlet命名为:UploadServlet,并且在web.xml中映射成/UploadServlet;并且在index.jsp页面中加入Form表单如下：

	<form action="UploadServlet" method="post"  
	        enctype="multipart/form-data">  
	        用户名:  
	        <input type="text" name="username">  
	        <br>  
	        密码:  
	        <input type="text" name="password">  
	        <br>  
	        上传文件  
	        <input type="file" name="file">  
	        <br>  
	        <input type="submit" value="Submit">  
	</form>  

而UploadServlet中的post方法中加入下列代码：

	response.setContentType("text/html");  
	response.setCharacterEncoding("UTF-8");  
	request.setCharacterEncoding("UTF-8");  
	PrintWriter out = response.getWriter();  
	BufferedReader bufferReader = new BufferedReader(new InputStreamReader(  
	        request.getInputStream()));  
	String line = "";  
	while ((line = bufferReader.readLine()) != null) {  
	    System.out.println(line);  
	}

然后通过打印出request流中的信息如下：
	------WebKitFormBoundarysmYDzdkAUIGlvggB  
	Content-Disposition: form-data; name="username"  

	1540547545@qq.com  
	------WebKitFormBoundarysmYDzdkAUIGlvggB  
	Content-Disposition: form-data; name="password"  

	terrylmay  
	------WebKitFormBoundarysmYDzdkAUIGlvggB  
	Content-Disposition: form-data; name="file"; filename="SoftwareCache.java"  
	Content-Type: application/octet-stream  

	package com.redmoon.forum.plugin.errorquery;  
	public class SoftwareCache extends ObjectCache {  
	    public String putSoftwareInfo(String type, int id) throws CacheException {  
	        SoftwareDao softwareDao = new SoftwareDao();  
	        result = softwareDao.getDescriptionByID(id, type);  
	        rmCache.putInGroup(group + id + "_" + type, group, result);  
	        System.out.println("返回的是从数据库中取出的数据");  
	        return result;  
	    }  
	}  

	------WebKitFormBoundarysmYDzdkAUIGlvggB--  

可以看到如果Form表单中使用了enctype=”multipart/form-data”定义的表单当提交表单时表单字段信息以及文件信息放入到request的流中进行提交。所以如果想要获取到上传文件的内容只能通过解析流的方式拿到。而通过request.getHeader这种方法拿到的只能是图中的Header里面的信息
并且是拿不到Request Payload中的某个字段的信息的.后面就是最蛋疼的事了.恐怕解析起来要花好长时间，当然强力解析肯定是不行的.毕竟要找出规律才有重用性

通过一天的学习，研究终于写出了一个文本文件，图片文件以及其他文件都能够上传的通用类,并且能够支持多文件上传.只要界面中能够进行多选的话，不说废话了.核心代码如下

	while (i != -1) {  
	        // 拿到该行数据  
	        String line = new String(buff, 0, i);  
	        if (line.startsWith("Content-Disposition:")) {  
	            // 如果不是文件域，那么肯定是表单字段域了  
	            if (!line.contains("filename")) {  
	                int index = line.indexOf("name=");  
	                String name = "";  
	                if (index != -1) {  
	                    name = line.substring(index + 6, line.length() - 3);  
	                    System.out.println(name);  
	                }  
	                // 找到字段域中的值  
	                in.readLine(buff, 0, 1024);  
	                i = in.readLine(buff, 0, 1024);  
	                String value = new String(buff, 0, i);  
	                // 放入到fileds中.到时候查找的时候也方便查找  
	                fields.put(name, value);  
	                System.out.println("(" + name + "," + value + ")");  
	            } else if (line.contains("filename")) {  
	                int index = line.indexOf("filename=");  
	                // 通过内容格式找到文件名称  
	                setFileName(line);  
	                System.out.println(this.fileName);  
	                // 连续读取2行，取出无用信息  
	                in.readLine(buff, 0, 1024);
	                in.readLine(buff, 0, 1024);  
	                // 读取文件的内容  
	                i = in.readLine(buff, 0, 1024);  
	                line = new String(buff, 0, i, "ISO8859_1");  
	                FileOutputStream out = new FileOutputStream(new File(  
	                        saveFilePath, fileName));  
	                while (i != -1 && !line.startsWith(this.boundary)) {  
	                    i = in.readLine(buff, 0, 1024);  
	                    if (new String(buff, 0, i).startsWith(boundary)) {  
	                        out.write(line.substring(0, line.length() - 2)  
	                                .getBytes("ISO8859_1"));  
	                        break;  
	                    } else  
	                        out.write(line.getBytes("ISO8859_1"));  
	                    line = new String(buff, 0, i, "ISO8859_1");  
	                }  
	            }  
	    }  
	    i = in.readLine(buff, 0, 1024);
	}

首先根据图中的头文件信息格式，来判断如果包含有filename信息的,则直接对后面的信息进行存储，如果不包含filename信息的，直接对其进行解析，然后把解析出来的字段以及他的value放入到map中.方便读取.
