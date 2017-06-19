---
layout: post
title: 如何在Amazon上搭建openfire服务器
date: 2016-02-21 12:18:00
tags: [Openfire,Amazon]
---

由于前段时间 通过本地搭建openfire服务器实现了客户端与Message的文本消息的通信，现在我想把服务器放在云端上运行，查了一下关于云服务器，包括阿里、腾讯 但是都比较贵，大约60-100元一个月吧..所以，转移目光发现，Amazon的EC2 可以免费试用一年的时间。

果断先试试Amazon的服务器。除了注册的时候填写银行卡 需要扣1美元的费用，以后这一年内，如果你没有添加新的服务项的话，这8块钱就足够了。而注册完成之后只需要根据Amazon的向导一步步的完成就可以了。最终启动一个Linux的实例。剩下的就是安装服务器以及环境配置方面相关的工作了。

通过ssh登录，进入到命令行系统。

一、安装JDK

使用如下命令安装openjdk-7-jre，java运行时环境

	sudo apt-get install openjdk-7-jre-headless

如果以后想使用javac编译.java文件的话，最好把jdk也安装一下。这个JDK

主要包括java的基础类、以及编译所需要的一些工具，比如jconsole、javac等等。使用如下命令安装jdk

	sudo apt-get install openjdk-7-jdk

在命令行中输入：

	java

验证是否安装完成。

在命令行中输入:

	javac

验证 java的编译器是否安装完成。

二、mysql 数据库的安装

在命令行中输入,检测更新

	sudo apt-get update

然后同时安装mysql服务器端 以及 客户端

	sudo apt-get install mysql-server mysql-client

在server安装过程中，会出现让你输入mysql-server的root账号的密码，这个密码是需要记住的。要不然后面没办法登录mysql-server了。
安装完成之后，使用

	mysql -u root -p

之后输入密码就可以进入到mysql的客户端命令模式下面了。

三、下载openfire的zip包

	wget -O openfire.tar.gz http://www.igniterealtime.org/downloadServlet?filename=openfire/openfire_3_9_3.tar.gz

下载的路径就是你当前命令行所在的目录。

然后我们需要把该gzip包解压。(后面解压的名称还得根据你本地的文件名来)

	tar -xzvf openfire_3_9_3.tar.gz

然后，将该解压之后的目录 移动到 另外一个目录下面

	mv openfire /usr/lib/

最后，需要开启openfire服务器。

	cd /usr/lib/openfire/bin/

调用openfire脚本

	./openfire start

这样，openfire服务器就算是搭建完成了。

四、为openfire创建数据库

使用如下命令

	mysql -u root -p

输入密码之后，进入mysql模式下

	mysql> create database openfire;

如果提示创建数据库成功，那么就不用管了。

后面就需要配置openfire服务器 与 mysql服务器的关联了。

五、配置openfire服务器

在配置openfire服务器之前，我们需要在amazon的管理控制台，找到正在运行的instance，并且进入instance的管理界面，找到安全组。将9090端口开放(添加到入站规则里面去，协议类型选择自定义TCP协议)

这样，我们通过Amazon分配给你的该主机的域名就可以 访问openfire的后台管理系统了。

http://ec2-52-68-82-92.ap-northeast-1.compute.amazonaws.com:9090/

注意，在openfire配置数据库的时候，选mysql，在选jdbc连接那一栏的hostname 只需要填写127.0.0.1即可，因为本身openfire服务器也是在那台amazon的系统上面。后面的admin的密码也是需要记住的。要不然进不去管理系统噢。
如果有客户端需要连接进来的话，我猜测还需要将5222、5223端口开放才可以。
