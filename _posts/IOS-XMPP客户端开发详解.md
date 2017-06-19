---
title: IOS XMPP客户端开发详解
date: 2016-02-21 11:59:49
tags: [XMPP, 报文]
---

由上篇文章将了如何在MAC本地搭建一个Openfire服务器，今天我们就要着手使用 XMPPFramework来开发 基于XMPP 协议的即时通讯IOS 客户端系统。今天主要看登录功能开发，可能有人会质疑，我们都没有开发注册功能，怎么开发登录功能。注册账号我们有捷径，服务器都在本地，当然很好做了。另外就是通过MAC 自带的客户端 Messages 进行注册。

<!-- more -->

首先，我们需要搭建起来IOS 的工程，并且将XMPPFramework 引入到我们的工程中来,当然我们可以使用源码 
直接导入，也可以使用cocoapods来进行依赖。本文直接使用源码来进行开发。由于在Github上的源码有一些问题，所以大家可以来https://github.com/TerryLMay/TMXMPPClient/tree/master/TMXMPPClient/ThirdTools/XMPPFrameworkio下载相关的XMPPFramework相关的代码。

下载关于XMPPFramework的代码之后,我们需要做的是 创建一个工程，然后将源代码导入到我们的工程中去。当然直接导入并且编译会出现libxml找不到的问题。我们需要在工程中导入libxml2.tbd(ios 9之后的名称) 以及 libresolv.9.tbd(ios 9之后的后缀)。然后在Build Setting的Head search 中加上 头文件的链接地址
${SDKROOT}/usr/include/libxml2

这之后，重新编译工程即可编译成功。后面，我们需要做的就是开发登录功能了，在开始之前，我们还是先看一下怎么注册一个账号吧；登录Openfire后台，创建一个用户。

通过在浏览器中输入 (前提 是 openfire服务器以及mysql服务器都已经启动了)

127.0.0.1:9090     

进入如下界面

<img src='/img/blogImages/XMPPLogin/OpenFireServerHomePage.png'/>

点击左上角用户/组 进入用户管理界面

<img src='/img/blogImages/XMPPLogin/OpenfireUserHomePage.png'/>

然后点击 左侧 导航栏中的新建用户 填写好用户信息就可以了。

<img src='/img/blogImages/XMPPLogin/OpenfireCreateUserPage.png'/>

到这边基本上用户注册环节 已经结束了。

开发 登录功能，基本上包括 登录界面的开发、XMPP登录逻辑的开发。登录界面我们可以 随便搭建一个登录界面就可以了。我今天主要是说一下 XMPP登录部分。

首先，创建关于XMPPLoginManager类，实现XMPPStreamDelegate，我定义了自己的一个宏，表示使用的Openfire服务器的地址 以及 端口号


	{% codeblock lang:objc %}	
	#define LocalOpenfire 1

	#if LocalOpenfire

	#define HOST_NAME @"127.0.0.1"
	#define HOST_PORT 5222
	#define CONNECT_IDENTIFIER @"@"

	#endif

	#define TIME_OUT 20
	
	//然后 定义XMPPStream、XMPPReconnect实例，并且初始化
	- (void)initXMPPStream {
	    self.loginXmppStream = [[XMPPStream alloc] init];
	    [self.loginXmppStream addDelegate:self delegateQueue:dispatch_get_main_queue()];
	}

	- (void)initXMPPReconnect {
	    self.loginReconnectManager = [[XMPPReconnect alloc] init];
	    [self.loginReconnectManager activate:self.loginXmppStream];
	    // You can also optionally add delegates to the module.
	    [self.loginReconnectManager addDelegate:self delegateQueue:dispatch_get_main_queue()];
	}

	//定义 Login按钮点击之后的点击事件调用 XMPPLoginManager中的如下方法进行连接：
	
	#pragma mark -- connect xmpp method for login viewController
	- (void)connectXMPPServer:(NSString *)userName password:(NSString *)password {
	    self.userName = userName;
	    self.password = password;

	    NSString *myJid = [NSString stringWithFormat:@"%@%@%@", userName, CONNECT_IDENTIFIER, HOST_NAME];
	    self.loginXmppStream.myJID = [XMPPJID jidWithString:myJid];

	    self.loginXmppStream.hostName = HOST_NAME;
	    self.loginXmppStream.hostPort = HOST_PORT;

	    NSError *connectError = nil;
	    [self.loginXmppStream connectWithTimeout:TIME_OUT error:&connectError];

	    if (connectError) {
	        NSLog(@"%@", connectError);
	        [self.loginDelegate loginXMPPConnectError:connectError];
	    }
	}

	//后面就是处理XMPPStream的各种回调就可以了

	#pragma mark -- xmppstream delegate
	//连接xmpp成功之后,使用密码认证
	- (void)xmppStreamDidConnect:(XMPPStream *)sender {

	    NSError *authError = nil;
	    [self.loginXmppStream authenticateWithPassword:self.password error:&authError];

	    if (authError) {
	        NSLog(@"%@", authError);
	        [self.loginDelegate loginXMPPDidNotAuthenticate];
	    }
	}

	//认证通过之后的处理
	- (void)xmppStreamDidAuthenticate:(XMPPStream *)sender {
	    NSLog(@"%@", @"认证通过");
	    [self.loginDelegate loginXMPPDidAuthenticate];
	}

	//连接服务器的超时处理
	- (void)xmppStreamConnectDidTimeout:(XMPPStream *)sender {
	    NSLog(@"连接超时");
	    [self.loginDelegate loginXMPPConnectDidTimeout];
	}

	//认证没有通过处理
	- (void)xmppStream:(XMPPStream *)sender didNotAuthenticate:(DDXMLElement *)error {
	    NSLog(@"认证失败");
	    [self.loginDelegate loginXMPPDidNotAuthenticate];
	}
	{% endcodeblock %}
	
基本上这样就可以登录了，登录成功之后，可以跳转到相应的界面。提一下我们 能看到的XMPP交互流程 包括：

	1、XMPP 使用用户名去连接服务器；
	2、XMPP 连接成功之后，通过密码去服务器认证
	3、认证成功之后，处理自己想处理的一下事情。

但其实XMPP 报文交互包括好几步，我就从别人那边摘录一点交互细节过来，就不自己一个个的抓包看了。
C1: 客户端初始化流给服务器 

	<stream:stream to="127.0.0.1" xmlns="jabber:client" 
		xmlns:stream="http://etherx.jabber.org/streams" version="1.0"> 

S1: 服务器向客户端发送流标签作为应答: 

	<?xml version='1.0' encoding='UTF-8'?><stream:stream 
		xmlns:stream="http://etherx.jabber.org/streams" xmlns="jabber:client" from="192.168.1.185" id="fb0cfcad" xml:lang="en" version="1.0"> 

S2: 发送 STARTTLS范围 

	<stream:features> 
	<starttls xmlns="urn:ietf:params:xml:ns: xmpp-tls"></starttls> <mechanisms xmlns="urn:ietf:params:xml:ns:xmpp-sasl"> 
	<mechanism>PLAIN</mechanism> <mechanism>CRAM-MD5</mechanism> <mechanism>ANONYMOUS</mechanism> <mechanism>DIGEST-MD5</mechanism> <mechanism>JIVE-SHAREDSECRET</mechanism> </mechanisms> 
	<compression xmlns="http://jabber.org/features/compress"> 
	<method>zlib</method> </compression> 
	<auth xmlns="http://jabber.org/features/iq-auth"/> 
	<register xmlns="http://jabber.org/features/iq-register"/> </stream:features>  

C2:客户端发送 STARTTLS 命令给服务器: 

	<starttls xmlns='urn:ietf:params:xml:ns:xmpp-tls'/> 

S3:服务器通知客户端可以继续进行: 

	<proceed xmlns='urn:ietf:params:xml:ns:xmpp-tls'/> 

C3：TLS 握手成功, 客户端初始化一个新的流给服务器

	<stream:stream to="192.168.1.185" xmlns="jabber:client" 
	xmlns:stream="http://etherx.jabber.org/streams" version="1.0">  

S4：服务器通知客户端可用的验证机制: 

	<?xml version='1.0' encoding='UTF-8'?> 
	<stream:stream xmlns:stream="http://etherx.jabber.org/streams" 
	xmlns="jabber:client" from="192.168.1.185" id="ad6f53e8" xml:lang="en" version="1.0"> <stream:features> 
	<mechanisms xmlns="urn:ietf:params:xml:ns:xmpp-sasl"> <mechanism>DIGEST-MD5</mechanism> <mechanism>PLAIN</mechanism> 
	<mechanism>ANONYMOUS</mechanism> <mechanism>CRAM-MD5</mechanism> </mechanisms> 
	<compression xmlns="http://jabber.org/features/compress"> <method>zlib</method> </compression> 
	<auth xmlns="http://jabber.org/features/iq-auth"/> 
	<register xmlns="http://jabber.org/features/iq-register"/> </stream:features> 

C4: 客户端选择一个验证机制: 

	<auth mechanism="DIGEST-MD5" xmlns="urn:ietf:params:xml:ns:xmpp-sasl"></auth> 

S5:服务器发送一个 [BASE64] 编码的挑战给客户端: 

	<challenge xmlns="urn:ietf:params:xml:ns:xmpp-sasl">cmVhbG09IjE5Mi4xNjguMS4xODUiLG5vbmNlPSJlcEJaZlBxU1p0WGlLYzBqdGpwT0I1a01HMHdiY0hsUmNhOE52ZE9SIixxb3A9ImF1dGgiLGNoYXJzZXQ9dXRmLTgsYWxnb3JpdGhtPW1kNS1zZXNz</challenge>  C5:客户端发送一个[BASE64]编码的回应这个挑战: 
	<response xmlns="urn:ietf:params:xml:ns:xmpp-sasl">Y2hhcnNldD11dGYtOCx1c2VybmFtZT0iYWRtaW4iLHJlYWxtPSIxOTIuMTY4LjEuMTg1Iixub25jZT0iZXBCWmZQcVNadFhpS2MwanRqcE9CNWtNRzB3YmNIbFJjYThOdmRPUiIsbmM9MDAwMDAwMDEsY25vbmNlPSJMNDJ1SE1XK2piemh6N1hzdWRndHN1V1VIT2hNZmFLVUJpcU5iR1p2IixkaWdlc3QtdXJpPSJ4bXBwLzE5Mi4xNjguMS4xODUiLG1heGJ1Zj02NTUzNixyZXNwb25zZT1hN2JhMWZlOThiMDc2ZjUzZWUzNTczM2Q5NDMwODJlYSxxb3A9YXV0aCxhdXRoemlkPSJhZG1pbiI=</response> 

S6:服务器通知客户端验证成功 

	<success xmlns="urn:ietf:params:xml:ns:xmpp-sasl">cnNwYXV0aD0yNDZlZDcyOTQ3ZjVhYzFiNWQ2ZDc4ZTkxM2QzMmFjMQ==</success> 

C6客户端初始化一个新流给服务器： 

	<stream:stream to="192.168.1.185" xmlns="jabber:client" 
	xmlns:stream="http://etherx.jabber.org/streams" version="1.0">

基本上XMPP的登录细节就已经清楚了。后面我们看一下 注册相关的模块 以及 联系人请求模块的开发。如果想获取源码的话，请转到https://github.com/TerryLMay/TMXMPPClient/tree/master/TMXMPPClient