---
layout: post
title: TCP TIME_WAIT状态之内核参数
date: 2018-09-04 22:30:00
tags: [网络, 握手, TCP]
---

### 前言

上篇中记录了关于TIME_WAIT产生的原因, 本篇主要记录LINUX 内核中跟TIME_WAIT相关的部分参数以及其含义


### TCP TIME_WAIT内核参数

1、net.ipv4.tcp_tw_reuse 参数
2、net.ipv4.tcp_tw_recycle 参数
3、net.ipv4.tcp_fin_timeout 参数
4、net.ipv4.tcp_timestamps 参数
5、tcp_max_tw_buckets 参数

下面就逐一理解这些参数的含义:

第一个参数: ```net.ipv4.tcp_tw_reuse``` 参数 表示开启重用。允许将TIME-WAIT sockets 重新用于新的TCP连接，默认为0，表示关闭;

该参数如果打开, 

第二个参数: ```net.ipv4.tcp_tw_recycle``` 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭;

第三个参数: ```net.ipv4.tcp_fin_timeout``` 表示TCP状态处于FIN_WAIT_2状态的超时时间;

第四个参数: ```net.ipv4.tcp_timestamps``` 

第五个参数: ```tcp_max_tw_buckets``` 表示系统允许存在的最多的处于TIME_WAIT状态的个数;



