---
layout: post
title: ubuntu下的一些操作
date: 2016-02-21 12:16:10
tags: [ubuntu]
---

问题一:如何修改Ubuntu的root密码

今天 注册了一个amazon的ec2 Ubuntu linux系统，因为 自己只需要在网页上面 配置一些选项，并且启动就可以了。所以，root用户以及密码 我还没有涉及修改。

但是，现在要修改一些东西需要 用到Ubuntu的root权限，但是 自己又不记得在输入 su之后 又不知道密码是什么。这时候就需要命令去重置一下root的密码了。

	sudo passwd root

再输入如下命令之后，terminal会有如下提示：

	Enter new UNIX password:
	Retype new UNIX password:

这时候，你只需要重新输入你的新密码就可以了。重置完成之后，输入su，根据提示 输入密码就能够进入root状态了。

	root@ip-172-31-17-27:/home/ubuntu#

这是比较实用的一招，记录下来以防以后用到。

问题二：命令行如何登录mysql

通过如下命令：

	mysql -u root -p

然后输入密码就可以了。
