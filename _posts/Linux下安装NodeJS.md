---
title: Linux下安装NodeJS
date: 2016-02-21 12:21:40
tags: [NodeJS]
---

前段时间在Amazon上申请了一台Ubuntu的Linux服务器，并且搭建了自己的Openfire服务器，同时，写了自己的XMPP客户端登录 注册，聊天的代码，实现了基本的通信。一台服务器总不能只开放一个端口来用，这样也太浪费了。果断想到搭建自己的Http服务器，如果在以前的话，二话不说先下载Tomcat，然后使用struts、Spring搭建自己的服务器。在以前 不懂 Java Servlet 或者 Php等后台开发的人 就很难搭建自己的服务器了。现在，NodeJS 成功解决了前端开发人员不懂后台的难题。NodeJS是基于 事件驱动机制、异步I/O的Javascript执行引擎。单线程，在IO密集型应用中表现出了很大的优势。今天我们就看一下如何在Linux系统下搭建NodeJS，以及NodeJS 模块管理 npm

<!-- more -->

第一种做法：(最简单)

一、安装NodeJS

	sudo apt-get install nodejs

二、安装npm

	sudo apt-get install npm

三、查看是否安装成功

	nodejs -v

四、尝试NodeJS 语法

	nodejs
	>console.log('HelloWorld');

就可以看到 在命令行中 打印出了 HelloWorld的字样。到这边NodeJS就安装成功了。
但是这种做法 造成的后果就是 在使用NodeJS的时候 命令都需要使用nodejs，感觉很不方便，我还是喜欢使用node -v这样的命令

第二种做法：(最难)

一、登录Linux终端

登陆到Linux终端，进入/usr/local/src目录，如下：

	root@ubuntu:~# cd /usr/local/src/

二、获取NodeJS源码

http://nodejs.org/dist/latest-v0.12.x/node-v0.12.10-linux-x64.tar.gz
目前最新版本的代码，你也可以去http://nodejs.org/dist/ 看一下有没有更新的版本。

三、解压 并 安装

	//解压
	 sudo tar xvf node-v0.12.10-linux-x64.tar.gz

	 安装
	 cd node-v0.12.10-linux-x64
	 cd bin
	 pwd    //找到当前所在目录/usr/local/src/node-v0.12.10-linux-x64/bin

	 //配置该Node的环境变量
	 export PATH=$PATH:/usr/local/src/node-v0.12.10-linux-x64/bin

如果要环境变量为永久存在,需要 vim /etc/profile 在该文件的最后一行加上

	//配置该Node的环境变量
	 export PATH=$PATH:/usr/local/src/node-v0.12.10-linux-x64/bin
	
四、查看是否安装成功

	node -v

五、尝试NodeJS语法

	node
	>console.log('HelloWorld');

六、安装NPM

	sudo apt-get install npm

