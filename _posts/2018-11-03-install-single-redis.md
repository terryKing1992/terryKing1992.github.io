---
layout: post
title: 深入理解Redis之Redis单机环境搭建
date: 2018-10-18 22:30:00
tags: [redis]
---

----------

>在实际开发项目过程中, 如果说要用到缓存, 那么第一个想到的一定是Redis, 但是为什么选Redis大多数人都不会去了解, 也不会去思考, 只知道它能当缓存使用, 比数据库快一点, 恰巧我也是这样的一个人;所以, 当我想写一篇关于Redis介绍的时候, 我竟然无从说起; 这也是对于Redis以及主流内存数据库不熟的原因; 不过, 在以后的日子里, 一定增加自己对于框架的思考与深入, 让自己在后面的技术道路上有所沉淀, 希望以后有人让我简要介绍Redis的时候, 我不会无从说起;这或许就是我想写Redis系列博客的目的所在吧! 

----------

## 一、Redis环境搭建 ##

### 下载redis稳定版 ###

```
curl -o redis.tar.gz http://download.redis.io/releases/redis-stable.tar.gz
```

### 解压redis包 ###

```
tar -zxvf redis-stable.tar.gz -C ./ // 该命令表示解压tar.gz包到当前目录
```

### 编译安装redis ###

创建redis的bin目录以及conf目录

```
sudo mkdir /usr/local/redis/bin
sudo mkdir /usr/local/redis/conf
```

进入到解压的Redis的目录下, 使用如下命令编译安装Redis

```
sudo make && make install PREFIX=/usr/local/redis
```

### 编辑配置Redis配置文件 ###

```
sudo cp redis.conf /usr/local/redis/conf/
```

### 启动Redis服务 ###

```
./redis-server ../conf/redis.conf  & //启动的时候后台运行
```

启动输出日志: 

```
45894:C 02 Nov 2018 22:11:19.922 # Redis version=5.0.0, bits=64, commit=00000000, modified=0, pid=45894, just started
45894:C 02 Nov 2018 22:11:19.922 # Configuration loaded
45894:M 02 Nov 2018 22:11:19.924 * Increased maximum number of open files to 10032 (it was originally set to 256).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 45894
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

45894:M 02 Nov 2018 22:11:19.933 # Server initialized
45894:M 02 Nov 2018 22:11:19.933 * Ready to accept connections
```

### 验证Redis服务 ###

使用网络工具`telnet`验证

```
terrydeMacBook-Air:bin terrylmay$ telnet 127.0.0.1 6379
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
```

使用系统进程`ps` 验证

```
terrydeMacBook-Air:bin terrylmay$ ps -ef | grep redis
  501 45894 44430   0 10:11下午 ttys000    0:00.04 ./redis-server 127.0.0.1:6379 //一个是Redis服务
  501 45897 44430   0 10:11下午 ttys000    0:00.00 grep redis //ps查询进程自己
```

到这里, 一个单机版的Redis服务就搭建完成了!

## 二、使用Redis存储数据 ##

### Redis CLI连接Redis服务 ###

```
terrydeMacBook-Air:bin terrylmay$ ./redis-cli
127.0.0.1:6379> 
127.0.0.1:6379> set name terrylmay
OK
127.0.0.1:6379> get name 
"terrylmay"
127.0.0.1:6379> 
```

到此, 我们可以使用Redis系统来存储数据字符串数据了.