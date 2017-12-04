---
layout: post
title: 本地如何搭建Angular2前端SPA框架
date: 2017-12-03 21:30:00
tags: [Angular2, Html, Type Script]
---

本博客主要记录一下在使用angular2过程中遇到的一些问题, 以及记录如何搭建一个angular2的前端应用. 同时如何 结合SpringBoot搭建前后台应用,本文主要使用angular-cli来创建Angular应用, 使用TypeScript来作为开发语言进行开发.

### 一、搭建NodeJS环境以及NPM工具

1、使用Homebrew安装NodeJS的环境

	$ brew install node

如果出现如下的输出, 则说明node已安装成功

![NodeJS安装成功](/assets/images/2017-12-04-install_node_using_brew.png)

2、验证NodeJS安装成功

	$ node -v //如果有类似输出: v8.9.1, 则说明已安装成功

3、使用Homebrew安装NPM包管理工具

	$brew install npm

如果有以下输出, 则说明NPM以安装成功
	
![NodeJS安装成功](/assets/images/2017-12-04-install_npm_using_brew.png)

4、验证NPM是否安装成功

	$ npm -v //如果出现npm command not found这种错误, 则需要手动配置一下npm所在的路径到PATH

	//编辑~/.bash_profile, 在文件末尾加上如下语句
	export PATH=${PATH}:/usr/local/opt/npm/libexec/bin

5、配置npm的镜像源为中国区淘宝镜像源

	$ npm install -g cnpm --registry=https://registry.npm.taobao.org

	//编辑~/.bash_profile, 在文件末尾加上如下语句
	export PATH=${PATH}:/usr/local/Cellar/node/8.9.1/bin



如果出现如下界面输出则说明cnpm安装成功

![安装cnpm输出](/assets/images/2017-12-04-install_cnpm_using_npm.png)

### 二、搭建Angular2环境

1、安装Angular-cli命令工具

	$ cnpm install -g angular-cli@latest

如果出现如下界面, 则说明angular/cli安装成功
![安装angular-cli结果](/assets/images/2017-12-04-install-angular-cli-using-cnpm.png)


2、使用angular-cli创建angular2应用

	$ ng new AwesomeAngular

如果出现如下输出, 则说明初始化工程成功

![初始化Angular工程成功](/assets/images/2017-12-04-init-angular-using-ng.png)

但是一般会卡在

	Installing packages for tooling via npm.

没关系, 我们使用Ctrl + C结束node_modules的下载, 然后在当前工程目录使用如下命令完成依赖的安装

    $cnpm install

一般我们会看到如下输出, 关于某一些包被放弃, 所以我们需要重新调整一下依赖

![Deprecate包](/assets/images/2017-12-04-deprecate-node-modules.png)

3、 运行如下命令调整关于一些deprecate的包

	$ cnpm install -S minimatch
	$ cnpm install -S @angular/cli

### 三、启动Angular项目

1、使用命令启动Angular Server

	cnpm start

启动命令会有如下输出：

![启动服务](/assets/images/2017-12-04-start-run-angular-server.png)

2、启动成功之后, 我们通过访问

	localhost:4200

查看程序运行结果

![界面](/assets/images/2017-12-04-ng-serve-result.png)

该文仅仅记录了如何搭建起来Angular应用, 后面会记录如何与SpringBoot整合完成一个简单的登录功能。
