---
layout: post
title: 如何让HexoServer 后台运行
date: 2016-02-21 15:26:16
tags: [Hexo, 后台运行]
---

在Hexo 工程目录下面创建一个app.js文件作为项目的启动文件,并将如下代码拷贝到文件中

	var spawn = require('child_process').spawn;             // 注册nodeJs子进程
	free = spawn('hexo', ['server', '-p 80']);              // 注入命令参数

	free.stdout.on('data', function (data) {
	    console.log('standard output:\n' + data);           // 捕获标准输出并将其打印到控制台
	});

	free.stderr.on('data', function (data) {
	    console.log('standard error output:\n' + data);     // 捕获标准错误输出并将其打印到控制台  
	});

	free.on('exit', function (code, signal) {
	    console.log('child process eixt ,exit:' + code);    // 注册子进程关闭事件
	});

安装forever 或者 pm2 来启动该 Hexo Server,就可以了。
