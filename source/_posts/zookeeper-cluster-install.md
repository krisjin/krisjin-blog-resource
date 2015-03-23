title: zookeeper集群安装
date: 2015-03-23 16:18:59
categories: zookeeper
tags: [zookeeper,分布式]
---

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，它包含一个简单的原语集，分布式应用程序可以基于它实现同步服务，配置维护和命名服务等。Zookeeper是hadoop的一个子项目，其发展历程无需赘述。在分布式应用中，由于工程师不能很好地使用锁机制，以及基于消息的协调机制不适合在某些应用中使用，因此需要有一种可靠的、可扩展的、分布式的、可配置的协调机制来统一系统的状态。Zookeeper的目的就在于此。本文简单分析zookeeper的工作原理，对于如何使用zookeeper不是本文讨论的重点。<!--more-->


## 下载Zookeeper安装包

去官网下载： [http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz](http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz "zookeeper-3.4.6.tar.gz ")

## 环境准备
准备三台ubuntu虚拟机hostname分别为：

- zookeeper01
- zookeeper02
- zookeeper03


## 安装
将下载后的文件解压

解压后修改配置文件zoo.cfg：

	# The number of milliseconds of each tick
	tickTime=2000
	# The number of ticks that the initial
	# synchronization phase can take
	initLimit=10
	# The number of ticks that can pass between
	# sending a request and getting an acknowledgement
	syncLimit=5
	# the directory where the snapshot is stored.
	# do not use /tmp for storage, /tmp here is just
	# example sakes.
	dataDir=/home/zookeeper/data
	# the port at which the clients will connect
	clientPort=2181
	# the maximum number of client connections.
	# increase this if you need to handle more clients
	#maxClientCnxns=60
	#
	# Be sure to read the maintenance section of the
	# administrator guide before turning on autopurge.
	#
	# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
	#
	# The number of snapshots to retain in dataDir
	#autopurge.snapRetainCount=3
	# Purge task interval in hours
	# Set to "0" to disable auto purge feature
	#autopurge.purgeInterval=1
	#server.A = B:C:D 
	server.1=zookeeper01:2888:3888
	server.2=zookeeper02:2888:3888
	server.3=zookeeper03:2888:3888

                 
配置文件说明：

- tickTime ：基本事件单元，以毫秒为单位。这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。

- dataDir ：存储内存中数据库快照的位置，顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。

- clientPort ：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

- initLimit：这个配置项是用来配置 Zookeeper 接受客户端初始化连接时最长能忍受多少个心跳时间间隔数，当已经超过 5 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒。

- syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒

- server.A = B:C:D : A表示这个是第几号服务器,B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader                



**创建数据目录**

	mkdir -p /home/zookeeper/data

**在标识Server ID.**

在/home/zookeeper/data目录中创建文件 myid 文件，每个文件中分别写入当前机器的server id，例如1.2.3.4这个机器，在/home/zookeeper/data目录的myid文件中写入数字  
1.



修改host文件：

	vi /etc/hosts
文件清单：

	127.0.0.1       localhost
	192.168.244.131  zookeeper01
	192.168.244.132  zookeeper02
	192.168.244.133  zookeeper03
	
	# The following lines are desirable for IPv6 capable hosts
	::1     ip6-localhost ip6-loopback
	fe00::0 ip6-localnet
	ff00::0 ip6-mcastprefix
	ff02::1 ip6-allnodes
	ff02::2 ip6-allrouters
                             
	
