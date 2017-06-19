---
layout: post
title: MAC下Openfire服务器的安装
date: 2016-02-21 11:27:33
tags: [Openfire]
---

这篇文章将要介绍如何在MAC 下安装Openfire服务器以及如何配置服务器，后续会实现 IOS 客户端 与 服务器端的通信。

一、Openfire的安装

1.到<a href = 'http://www.igniterealtime.org/downloads/index.jsp'>http://www.igniterealtime.org/downloads/index.jsp</a> 下载最新openfire for mac版
2、下载完按照正常mac程序安装即可。
3、完成安装之后，进入到设置界面，打开最下面的openfire图标，打开openfire的打开界面。
4、在点击start open fire的时候会出现 could not start openfire 服务器。这时候，需要检查自己是否安装了Java SDK，以及配置了JAVA_HOME环境变量。
5、当你再次尝试 去 start openfire server的时候可能还是打开失败,这时候按照如下命令操作即可。

	sudo chmod -R 777 /usr/local/openfire

6、好像设置界面的openfire的start openfire server并不管用，需要cd 到 /usr/local/openfire/bin下面，运行

	$./openfire.sh

可以看到如下输出，就说明openfire服务器已经开启了。

这时候我们先不着急 去开发IOS 的 即时通信客户端。先用现有的客户端进行测试。推荐 mac自带的message客户端以及目前mac下比较流行的客户端aidum来测试。注册用户的话可以在管理控制台添加，也可以使用客户端注册。
然后就可以看两个客户端之间怎么通信了。
