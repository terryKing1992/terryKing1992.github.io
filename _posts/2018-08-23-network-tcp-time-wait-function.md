---
layout: post
title: TCP TIME_WAIT状态之作用
date: 2018-09-07 22:30:00
tags: [网络, 握手, TCP]
---

### 前言

上篇中记录了关于TIME_WAIT产生的原因, 本篇主要记录TIME_WAIT的作用; 

### TIME_WAIT存在的作用

主动关闭方: A
被动关闭方: B

上篇我们知道A在发送完对于B的FIN报文的确认报文ACK之后, 会进入TIME_WAIT状态; 但是网络环境会存在丢包的情况, 那么当主动关闭一方对于被动关闭一方的FIN确认报文ACK在网络中丢失的时候。为了能够让客户机服务器两遍的连接最终都处于CLOSED状态来释放文件数以及网络连接标识资源, 就需要有TIME_WAIT这种机制来保证。

那么我们看一下TIME_WAIT是如何保证两边都能够进入CLOSED状态的; 我们从四个场景来看:

场景一、A发送给B的ACK确认报文正确到达B

这个时候没有什么问题, 处于LAST_ACK状态的B收到A的确认报文之后, 直接进入CLOSED状态, 而 A经过 2MSL的等待之后, 进入CLOSED状态

场景二、A发送给B的ACK确认报文没有到达B, A处于TIME_WAIT状态

这个时候B经过一段时间(这个一段时间应该是1MSL, 待确认)没有收到A的确认报文ACK, 那么B会重新发送FIN报文给A, 这个时候因为A的TIME_WAIT状态会持续 2MSL, 所以后面也会有两种情况: 

```
    1、在TIME_WAIT时间范围内, A收到了B重发的FIN报文, 那么A会回一个ACK给到B, 并不确保ACK一定到B; 等到TIME_WAIT超时之后, 进入CLOSED状态.
    2、在TIME_WAIT时间范围内, A并没有收到B重发的FIN报文, 那么A就进入CLOSED状态, 但是B还处于LAST_ACK状态, 那么经过 2个MSL之后(这里为什么是两个MSL, 理论上应该是自己发送的FIN + A返回的ACK报文 在网络中存在的最大时长, 待确认), 会再发送FIN给A, 因为此时A已经处于CLOSED状态, 所以A会发送给B一个RST报文, 来表示我已经断开连接了, 你也断开吧; 如果B一直收不到任何反馈, 那么会一直定时发送FIN给A 直到TCP的重传超时, B会由LAST_ACK状态进入CLOSED状态;
    3、在TIME_WAIT时间范围内, A并没有收到B重发的FIN报文, 那么A就进入CLOSED状态, 但是B还处于LAST_ACK状态, 如果这时候A又发起了SNY连接, 使用了上一次的端口号, 那么B收到SYN连接请求之后, 发现该连接处于LAST_ACK状态, 那么会发送RST给客户端, 导致三次握手失败;
```

 注: 1、对于网上某一篇博客的思考,博客中提到:当客户端开启net.ipv4.tcp_tw_reuse这个属性的时候, 会出现服务端发送的FIN报文, 被新的连接接收到从而导致断开连接, 这种情况看来是不存在的; 因为服务器处于LAST_ACK的时候, 即使开启tw_reuse, 也会被服务器给reset掉, 并不会重新建立连接的. 仅为个人理解, 如有不同观点欢迎添加微信: wangchao932963710 讨论;

注: 2、对于上面思考的重新思考: 当B在超时时间内没有收到ACK的时候, 那么B会重新发送FIN, 那么此时的LAST_ACK会不会重新开始计算超时时间? 如果重新开始计算超时时间, 那么注①的理解就是正确的, 如果不会重新开始计算LAST_ACK状态的超时时间, 确实有可能B发送完FIN之后, B马上LAST_ACK超时, B的状态变为了CLOSED状态, 此时客户端重新再同一端口上面发起SYN请求, B接受请求并建立连接; 如果此时FIN才到达A端, 那么A端会如何处理呢? 

1、根本不存在这种情况 2、如果丢弃, 是依托与什么判断机制来丢弃的, 3、如果断开新的连接, 那么TCP就是不可靠的传输协议了. 显然后面这种是不可能的, 所以只有去资料中查找什么机制去丢弃FIN报文了? 这个答案待后面查找资料补充.



