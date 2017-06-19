---
title: 如何通过subl打开文本文件
date: 2016-02-21 11:22:11
tags: [sublime]
---

如何通过命令行打开某一个文本文件
	$cd ~
	$open .bash_profile

将 alias subl=\’’/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl’\’

加入到配置文件的最后一行,并保存
重新打开命令行,然后 使用命令行

	$subl .bash_profile

看一下sublime 是不是把你的配置文件打开了.

如果没法打开的话，看一下 你应用程序下的sublime叫什么名字,比如Sublime Text 2 就需要把 上面的改成 alias subl=\’’/Applications/Sublime Text 2.app/Contents/SharedSupport/bin/subl’\’