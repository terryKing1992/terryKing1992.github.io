---
layout: post
title: XMPP好友获取与好友关系详解
date: 2016-02-21 12:14:16
tags: [XMPP]
---

上篇主要讲解了关于XMPP登录部分，而登录部分 只用到了XMPPStream 连接，认证。而本篇 主要讲解 用户的注册(XMPPStream)、自己上线通知(XMPPPresence)、好友获取(IQ 请求)。


<!-- more -->
---

用户注册部分

---

用户注册部分也比较简单。首先，我们需要自定义一个用户注册界面，界面的开发 就不在这边详细讲解了，github上面有开源的代码，你们可以去下载。xmpp的注册部分跟登录部分基本流程是相同的，xmpp注册 也需要先连接XMPP 服务器，当连接成功之后，需要调用XMPPStream的registe方法。我们来看一下具体实现：

首先，跟登录类方法一样，我们需要先初始化一下xmppStream实例

	- (void)initXMPPStream {
	    self.registeXmppStream = [[XMPPStream alloc] init];
	    [self.registeXmppStream addDelegate:self delegateQueue:dispatch_get_main_queue()];
	}

同时，使用本地已经创建的xmppStream 去发送连接服务器的请求

	#pragma mark -- register user
	- (void)registeNewUserFromXMPPServer:(NSString *)userName password:(NSString *)password {
	    self.userName = userName;
	    self.password = password;

	    NSString *myJid = [NSString stringWithFormat:@"%@%@%@", userName, CONNECT_IDENTIFIER, HOST_NAME];
	    self.registeXmppStream.myJID = [XMPPJID jidWithString:myJid];

	    self.registeXmppStream.hostName = HOST_NAME;
	    self.registeXmppStream.hostPort = HOST_PORT;

	    NSError *connectError = nil;
	    [self.registeXmppStream connectWithTimeout:TIME_OUT error:&connectError];
	    if (connectError) {
	        NSLog(@"%@", connectError);
	    }
	}
	其实，从上面的代码我们可以看出，这写代码跟登录的代码一模一样。后面才会看到不一样的地方
	//xmpp连接成功之后，发送注册请求
	- (void)xmppStreamDidConnect:(XMPPStream *)sender {
	    NSError *registeError;
	    [self.registeXmppStream registerWithPassword:self.password error:&registeError];
	}

	//注册成功之后，回调处理其他事情，比如跳转登录界面
	- (void)xmppStreamDidRegister:(XMPPStream *)sender {
	    [self.registeXMPPDelegate registeDidSuccess];
	}

	//注册失败之后应该如何处理
	- (void)xmppStream:(XMPPStream *)sender didNotRegister:(NSXMLElement *)error {
	    NSError *registerError = [NSError errorWithDomain:@"regist xmpp" code:-1 userInfo:@{@"error":error.description}];
	    [self.registeXMPPDelegate registeDidFailed:registerError];
	}

我们可以看到，登录模块 是在xmppStreamDidConnect的回调中 发起密码认证的报文，而在注册模块中，xmppStream连接成功之后，发起注册的报文，只有这一点不同而已。所以理解用户注册流程 也是非常容易的。
当注册成功之后，我们就可以用我们刚才注册的用户名 以及密码去登录了。

---
获取联系人模块
---

当我们登录成功之后，因为服务器并不会自动的下发给好友你的上线信息，需要自己发送一个报文 去告诉服务器 并且让服务器 转发给自己好友 自己的上线信息。所以，在登录成功之后，我们需要发送一个空的XMPPPresence，去告诉其他好友 我已经上线了。

	+ (void)sendOnLinePresence:(XMPPStream *)xmppStream {
	    //如果没有类型，默认类型为available
	    XMPPPresence *onLinePresence = [XMPPPresence presence];
	    [xmppStream sendElement:onLinePresence];
	}

发送了这条Presence 出席消息给服务器之后，服务器会转发给你的好友，你好友的客户端会受到Presence报文，类型为available。好友就可以更新你的在线状态了。
发送了上线报文之后，我们需要去同步我们的好友信息，这时候我们还需要发送一个IQ请求，去请求好友列表。当然我们也可以初始化XMPPRoster 通过这个类去获取好友列表。本篇就不这样做了，我们还是需要了解一下xmpp 报文的详细信息的。

	/*
	<iq type="get">
	   <query xmlns="jabber:iq:roster"/>
	</iq>
	*/

	+ (void)sendFetchRosterIQ:(XMPPStream *)xmppStream {
	    NSXMLElement *query = [NSXMLElement elementWithName:@"query" xmlns:@"jabber:iq:roster"];

	    XMPPIQ *iq = [XMPPIQ iqWithType:@"get" elementID:[xmppStream generateUUID]];
	    [iq addChild:query];

	    [xmppStream sendElement:iq];
	}

这样，我们通过这个方法 发送了请求好友的报文，服务器会返回给我们 好友信息。

	<iq type="result" id="F499F755-A8FA-4ED0-A839-185F0325A3A4" to="test1@127.0.0.1/tvxw6qoij"><query xmlns="jabber:iq:roster"><item jid="test95" name="terry" subscription="both"/><item jid="test92" name="maylor" subscription="both"/></query></iq>

我们可以从上面服务器返回的报文中看到，我的好友有两个test95 和 test92. 名字也有。这样就完成了用户注册，上线报文发送 以及 好友获取模块的开发。

在这里，其实我还想说一下联系人之间的关系，联系人 除了 上面报文里面 subscription=”both” 其实还有其他 几种格式。

	1、Both：表示 我们是双向好友；即 我订阅了test92，当test92上线的时候，服务器会告诉我，test92已经上线。同时test92订阅了我，即当我上线的时候，我发送的上线报文，服务器会帮我转发给test92
	2、to:表示我订阅了对方test92，即当test92上线的时候，test92发送上线报文给服务器，服务器会告诉我，test92上线了。但是我上线之后，服务器并不会告诉test92我上线了。
	3、from:与to正好相反。
	4、none：表示双方都没有相互订阅。即不是好友关系。

所以，很多人都说xmpp 报文比较重载，除了他登录与xmpp服务器交互多，xml 无用信息 多。而且包括 这一系列的冗余交互。本来 只需要一次交互就可以了。比如：我加你好友，你同意了。这样我们就是双向好友了，你删我了我们就变成单向好友了to。但是xmpp 必须是 你加(订阅)我了，我同意了。同时我加(订阅)你了，你也同意了，这样我们才是双向好友。
我也是醉了。如果要看最新的代码，请转到https://github.com/TerryLMay/TMXMPPClient/tree/master/TMXMPPClient
