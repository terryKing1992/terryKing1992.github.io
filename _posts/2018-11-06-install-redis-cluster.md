---
layout: post
title: 深入理解Redis系列之集群环境搭建
date: 2018-11-06 22:30:00
tags: [redis集群模式]
---

>前面分别写了关于单机版Redis搭建以及使用SpringBoot来访问Redis服务, 后面也就顺着写一些关于分布式环境的搭建以及高可用的大概原理;

## 分布式环境准备 ##

因为在前面一篇[深入理解Redis系列之单机Redis环境搭建](https://www.jianshu.com/p/608fcdf84909)中已经安装好了Redis的介质, 后面只需要使用配置文件指定不同端口来做Master以及Slave节点即可;因为是在一台机器安装, 所以只能通过端口来启动不同角色的Redis进程; 规划如下:

| 机器IP | 端口 | 角色 | 配置文件目录 |
| ------ | ------ | ------ | ------ |
|127.0.0.1 | 6379| Master1| /usr/local/redis/conf/6379/redis.conf |
|127.0.0.1 | 6380| 6380| /usr/local/redis/conf/6380/redis.conf |
|127.0.0.1 | 6381| 6381| /usr/local/redis/conf/6381/redis.conf |
|127.0.0.1 | 26379| 26379| /usr/local/redis/conf/26379/redis.conf |
|127.0.0.1 | 26380| 26380| /usr/local/redis/conf/26380/redis.conf |
|127.0.0.1 | 26381| 26381| /usr/local/redis/conf/26381/redis.conf |

`注: 本来使用的16379、16380、16381作为slave节点的端口的, 但是发现每次启动6379节点的时候总是把16379端口也同时占用, 经查证发现每一个redis集群的节点需要开通两个TCP端口。一个是用于客户端的Redis TCP，如6379。另一个由客户端加10000所得，如16379，用于Redis集群总线连接； 故改为26379、26380、26381`

## 生成配置文件 ##

使用如下配置可以批量修改与端口相关内容

```
vi redis.conf
esc 命令之后
:%s/6379/{replace_port}/g
```

分别修改如下配置: 
```
port 6379 //根据上面的规划调整
daemonize yes // 从no => yes
pidfile /var/run/redis_6379.pid //根据规划调整 
cluster-enabled yes //将该行注释去掉, 如果没有这一行就添加到配置文件
bind 127.0.0.1 //如果是外部访问，改为本机的IP地址，而不是本地回路127.0.0.1
dbfilename dump_6379.rdb //根据端口设置
logfile "6379.log" //启动日志也可以打开, 便于看出错日志

```

## 启动master与slave进程

```
cd /usr/local/redis/conf/6379
../../bin/redis-server redis.conf
cd /usr/local/redis/conf/6380
../../bin/redis-server redis.conf
cd /usr/local/redis/conf/6381
../../bin/redis-server redis.conf
cd /usr/local/redis/conf/26379
../../bin/redis-server redis.conf
cd /usr/local/redis/conf/26380
../../bin/redis-server redis.conf
cd /usr/local/redis/conf/26381
../../bin/redis-server redis.conf
```
我们可以通过`ps -ef | grep redis` 验证 进程是否起来:

```
 501 82074     1   0 12:08上午 ??         0:00.11 ../../bin/redis-server 127.0.0.1:6379 [cluster] 
  501 82077     1   0 12:08上午 ??         0:00.09 ../../bin/redis-server 127.0.0.1:6380 [cluster] 
  501 82079     1   0 12:08上午 ??         0:00.07 ../../bin/redis-server 127.0.0.1:6381 [cluster] 
  501 82082     1   0 12:08上午 ??         0:00.05 ../../bin/redis-server 127.0.0.1:26379 [cluster] 
  501 82085     1   0 12:08上午 ??         0:00.04 ../../bin/redis-server 127.0.0.1:26380 [cluster] 
  501 82087     1   0 12:08上午 ??         0:00.02 ../../bin/redis-server 127.0.0.1:26381 [cluster] 
  501 82090 81954   0 12:08上午 ttys003    0:00.00 grep redis

```

## 将Redis各进程加入到Redis集群 ##

1、使用如下命令启动集群:

```
redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:26379 127.0.0.1:26380 127.0.0.1:26381 --cluster-replicas 1
```

启动日志输出如下:

```
terrydeMacBook-Air:bin terrylmay$ ./redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:26379 127.0.0.1:26380 127.0.0.1:26381 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:26379 to 127.0.0.1:6379
Adding replica 127.0.0.1:26380 to 127.0.0.1:6380
Adding replica 127.0.0.1:26381 to 127.0.0.1:6381
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 3dbb14f3561c413a29c79eb75b39c670692bb1c1 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
M: d0fc2996f417ac47d4ca174817d7485148534f97 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
M: 2dfe9608f3a3fc131f746a3336aa2e01cfb11faa 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
S: 5594d0d4e6114533c4b598307a94eeb56605970f 127.0.0.1:26379
   replicates d0fc2996f417ac47d4ca174817d7485148534f97
S: 8d6edc0b4d95b989d94e12d9d7915b45db7a6a0b 127.0.0.1:26380
   replicates 2dfe9608f3a3fc131f746a3336aa2e01cfb11faa
S: 493d5dfe6d6aec9ce629dc55de58d4072ad1be93 127.0.0.1:26381
   replicates 3dbb14f3561c413a29c79eb75b39c670692bb1c1
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.....
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: 3dbb14f3561c413a29c79eb75b39c670692bb1c1 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 8d6edc0b4d95b989d94e12d9d7915b45db7a6a0b 127.0.0.1:26380
   slots: (0 slots) slave
   replicates 2dfe9608f3a3fc131f746a3336aa2e01cfb11faa
S: 5594d0d4e6114533c4b598307a94eeb56605970f 127.0.0.1:26379
   slots: (0 slots) slave
   replicates d0fc2996f417ac47d4ca174817d7485148534f97
S: 493d5dfe6d6aec9ce629dc55de58d4072ad1be93 127.0.0.1:26381
   slots: (0 slots) slave
   replicates 3dbb14f3561c413a29c79eb75b39c670692bb1c1
M: d0fc2996f417ac47d4ca174817d7485148534f97 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 2dfe9608f3a3fc131f746a3336aa2e01cfb11faa 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
terrydeMacBook-Air:bin terrylmay$ 
```

到此, Master-Slave集群已经搭建成功了; 

`注:刚开始没发觉, 后面验证集群master切换的时候才发现, 原来创建集群的时候并不是按顺序一对一的关系, 所以我们可以看到 127.0.0.1:26381 这个进程是 127.0.0.1:6379节点的Slave节点, 这也就是后面再启动6379进程的时候日志是连接到26381 master节点了;`


那么集群master节点与slave节点都有主从复制的功能, 下面我们来验证一下, 首先登录redis任意主节点，放入内容之后, 登录从节点

`./redis-cli -c -h 127.0.0.1 -p 6380` 

```
terrydeMacBook-Air:bin terrylmay$ ./redis-cli -c -h 127.0.0.1 -p 6380
127.0.0.1:6380> set name 123456
OK
127.0.0.1:6380> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=26379,state=online,offset=110378,lag=0
```
`./redis-cli -c -h 127.0.0.1 -p 26379`登录从节点, 发现通过name也可以拿到值

```
terrydeMacBook-Air:bin terrylmay$ ./redis-cli -c -h 127.0.0.1 -p 26379
127.0.0.1:26379> get name
-> Redirected to slot [5798] located at 127.0.0.1:6380
"123456"
127.0.0.1:6380> 
```

通过上面日志，我们看到该信息redirected到了master节点, 那么问题来了? 从slave里面读取数据，会再走到master中读数据么? 答案应该不会, 那样就没人用slave来当本地缓存了. 但是为什么不会呢? 还是想要了解一下! 放在后面探索吧;

这时候，我们删除其中的一个master 6379节点，看一下会发生什么情况?

```
 ps -ef | grep :6379 | awk '{print $2}' |xargs  kill -9  
```
我们在master6379的从节点26379节点上可以看到日志输出如下:

```
82082:S 05 Nov 2018 22:56:08.343 * FAIL message received from 2dfe9608f3a3fc131f746a3336aa2e01cfb11faa about 3dbb14f3561c413a29c79eb75b39c670692bb1c1
82082:S 05 Nov 2018 22:56:08.345 # Cluster state changed: fail
82082:S 05 Nov 2018 22:56:09.153 # Cluster state changed: ok
```

这时候, 我们还看不出master节点切换了, 仅仅看着像是切换了;

那么, 我们使用`redis-cli`命令进入到`redis控制台`

```
./redis-cli -c -h 127.0.0.1 -p 6380
```

使用命令`cluster nodes` 查看节点状态，可以看到
```
127.0.0.1:6380> cluster nodes
8d6edc0b4d95b989d94e12d9d7915b45db7a6a0b 127.0.0.1:26380@36380 slave 2dfe9608f3a3fc131f746a3336aa2e01cfb11faa 0 1541431115317 5 connected
493d5dfe6d6aec9ce629dc55de58d4072ad1be93 127.0.0.1:26381@36381 master - 0 1541431115000 7 connected 0-5460
3dbb14f3561c413a29c79eb75b39c670692bb1c1 127.0.0.1:6379@16379 slave 493d5dfe6d6aec9ce629dc55de58d4072ad1be93 1541431110864 1541431109000 7 disconnected
5594d0d4e6114533c4b598307a94eeb56605970f 127.0.0.1:26379@36379 slave d0fc2996f417ac47d4ca174817d7485148534f97 0 1541431116330 4 connected
2dfe9608f3a3fc131f746a3336aa2e01cfb11faa 127.0.0.1:6381@16381 master - 0 1541431115000 3 connected 10923-16383
d0fc2996f417ac47d4ca174817d7485148534f97 127.0.0.1:6380@16380 myself,master - 0 1541431115000 2 connected 5461-10922
```

6379节点已经处于`disconnected`状态了

使用命令`cluster info `查看集群状态, 发现cluster 状态正常

```
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
```
说明master节点已经切换到了26381节点;

然后, 我们试着重启刚才删掉的6379节点，可以看到6379日志打印如下:
```
90819:C 05 Nov 2018 23:05:56.067 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
90819:C 05 Nov 2018 23:05:56.067 # Redis version=5.0.0, bits=64, commit=00000000, modified=0, pid=90819, just started
90819:C 05 Nov 2018 23:05:56.067 # Configuration loaded
90820:M 05 Nov 2018 23:05:56.070 * Increased maximum number of open files to 10032 (it was originally set to 256).
90820:M 05 Nov 2018 23:05:56.073 * Node configuration loaded, I'm 3dbb14f3561c413a29c79eb75b39c670692bb1c1
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in cluster mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 90820
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

90820:M 05 Nov 2018 23:05:56.077 # Server initialized
90820:M 05 Nov 2018 23:05:56.077 * DB loaded from disk: 0.000 seconds
90820:M 05 Nov 2018 23:05:56.077 * Ready to accept connections
90820:M 05 Nov 2018 23:05:56.079 # Configuration change detected. Reconfiguring myself as a replica of 493d5dfe6d6aec9ce629dc55de58d4072ad1be93
90820:S 05 Nov 2018 23:05:56.079 * Before turning into a replica, using my master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
90820:S 05 Nov 2018 23:05:56.080 # Cluster state changed: ok
90820:S 05 Nov 2018 23:05:57.088 * Connecting to MASTER 127.0.0.1:26381
90820:S 05 Nov 2018 23:05:57.096 * MASTER <-> REPLICA sync started
90820:S 05 Nov 2018 23:05:57.097 * Non blocking connect for SYNC fired the event.
90820:S 05 Nov 2018 23:05:57.097 * Master replied to PING, replication can continue...
90820:S 05 Nov 2018 23:05:57.098 * Trying a partial resynchronization (request 385e7338e98550e9dc9d1cd5a3ea77d4e39e5f51:1).
90820:S 05 Nov 2018 23:05:57.099 * Full resync from master: 306b56738fb37bd1d1fcb9e0f709584c56f2df52:107352
90820:S 05 Nov 2018 23:05:57.099 * Discarding previously cached master state.
90820:S 05 Nov 2018 23:05:57.188 * MASTER <-> REPLICA sync: receiving 178 bytes from master
90820:S 05 Nov 2018 23:05:57.189 * MASTER <-> REPLICA sync: Flushing old data
90820:S 05 Nov 2018 23:05:57.189 * MASTER <-> REPLICA sync: Loading DB in memory
90820:S 05 Nov 2018 23:05:57.189 * MASTER <-> REPLICA sync: Finished with success
```

`Connecting to MASTER 127.0.0.1:26381` 说明先前master节点切换到26381节点上; 集群信息都有了更新; 当然我们可以通过`./redis-server -c -h 127.0.0.1 -p 26381` 然后使用 `info replication`查看slave节点信息

```
terrydeMacBook-Air:bin terrylmay$ ./redis-cli -c -h 127.0.0.1 -p 26381
127.0.0.1:26381> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6379,state=online,offset=108430,lag=0
master_replid:306b56738fb37bd1d1fcb9e0f709584c56f2df52
master_replid2:37b3866dc3956911c64eca1ea3df03aa786d4813
master_repl_offset:108430
second_repl_offset:107353
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:108430
```
到这里, 基本上主从切换就OK了; 下一篇文章写一下如何使用SpringBoot访问Redis集群;