title: redis install
date: 2015-03-19 09:20:44
categoris: NoSQL
tags: [nosql,redis]
---

Redis 是一个高性能的key-value数据库，属于NoSQL的一种。它提供了Python，Ruby，Erlang，PHP客户端，使用很方便。它跟memcached类似，不过数据可以持久化，而且支持的数据类型更丰富。有字符串，链表，集合和有序集合。支持在服务器端计算集合的并，交和补集(difference)等，还支持多种排序功能。<!--more-->


## 安装

	root@zookeeper03:/opt/redis-2.8.19# wget http://download.redis.io/releases/redis-2.8.19.tar.gz
	root@zookeeper03:/opt/redis-2.8.19# tar xvfz redis-2.8.19.tar.gz
	root@zookeeper03:/opt/redis-2.8.19# cd redis-2.8.19#
	root@zookeeper03:/opt/redis-2.8.19# make

编译完成后就会生成相关的二进制可执行文件，为了便于管理。创建3个目录分别用来存放可执行二进制文件、配置文件、数据持久化文件。将src目录下的二进制文件redis-benchmark、redis-check-aof、redis-check-dump、redis-cli、redis-sentinel（非必需）、redis-server拷贝到bin目录，配置文件redis.conf、sentinel.conf（非必需）拷贝到etc目录。

	root@zookeeper03:/opt/redis-2.8.19# mkdir -p /usr/local/redis/{bin,db,etc}
	root@zookeeper03:/opt/redis-2.8.19# cp src/redis-server /usr/local/redis/bin
	root@zookeeper03:/opt/redis-2.8.19# cp src/redis-benchmark /usr/local/redis/bin
	root@zookeeper03:/opt/redis-2.8.19# cp src/redis-check-aof /usr/local/redis/bin
	root@zookeeper03:/opt/redis-2.8.19# cp src/redis-check-dump /usr/local/redis/bin
	root@zookeeper03:/opt/redis-2.8.19# cp src/redis-cli /usr/local/redis/bin
	root@zookeeper03:/opt/redis-2.8.19# cp src/redis-sentinel /usr/local/redis/bin
	root@zookeeper03:/opt/redis-2.8.19# cp redis.conf /usr/local/redis/etc/
	root@zookeeper03:/opt/redis-2.8.19# cp sentinel.conf /usr/local/redis/etc

redis-server：Redis服务器的daemon启动程序，对应的默认配置文件redis.conf。  
redis-benchmark：Redis性能测试工具。  
redis-cli：Redis命令行操作工具，类似于mysql的控制台。  
redis-check-aof/redis-check-dump：修复损坏的数据文件file.aof/dump.rdb。  
redis-sentinel：用于管理多个Redis服务器（instance），为集群提供监控、提醒、自动故障迁移服务，对应的配置文件sentinel.conf。   


###### 调整默认的相关参数

	#以守护进程的模式运行实例
	daemonize yes
	#指定日志文件的位置。默认为空，以守护进程运行时日志默认会发送到/dev/null设备，不以守护进程运行时日志输出到标准输出设备。
	logfile "/var/log/redis.log"
	#指定存放数据库文件的文件夹
	dir "/usr/local/redis/db"


启动redis，可以看到redis默认监听TCP的6379端口

**./redis-server ./etc/redis.conf**



	root@zookeeper03:/opt/redis-2.8.19# netstat -tpln
	Active Internet connections (only servers)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      1277/mysqld     
	tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      9308/redis-server *
	tcp        0      0 127.0.1.1:53            0.0.0.0:*               LISTEN      1444/dnsmasq    
	tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      3231/cupsd      
	tcp6       0      0 :::6379                 :::*                    LISTEN      9308/redis-server *
	tcp6       0      0 ::1:631                 :::*                    LISTEN      3231/cupsd      


查看redis的日志时发现在启动的时候抛出了几条条警告：

	[9390] 18 Mar 19:11:47.121 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
	[9390] 18 Mar 19:11:47.141 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
	[9390] 18 Mar 19:11:47.141 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.



根据警告提示我们需要将内核参数vm.overcommit_memory设置为1，net.core.somaxconn设置成更大的数值，默认是128。编辑文件/etc/sysctl.conf，追加2行

	#sysctl文件追加的参数
	vm.overcommit_memory = 1
	net.core.somaxconn = 1024

保存退出后，刷新系统内核参数，运行命令：sysctl -p



关于这两个内核参数的解释：
vm.overcommit_memory
指定了内核针对内存分配的策略，其值可以是0、1、2。

0， 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。  

1， 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。

2， 表示内核允许分配超过所有物理内存和交换空间总和的内存。
net.core.somaxconn
定义了系统中每一个端口最大的监听队列的长度,这是个全局的参数,默认值为128，在某些应用下可能会限制接收新TCP连接侦听队列的大小。

4、用redis提供的命令行工具进行一些简单的操作


**用redis提供的命令行工具进行一些简单的操作**

	root@zookeeper03:/usr/local/redis/bin# ./redis-cli
	127.0.0.1:6379>
	#查看redis的各种信息参数，内存、连接数、键值数等
	127.0.0.1:6379> info
	#查看所有的key值
	127.0.0.1:6379> keys *
	(empty list or set)
	#插入并查看一个key值
	127.0.0.1:6379> set i 123
	OK
	127.0.0.1:6379> get i
	"123"
	#list操作
	127.0.0.1:6379> sadd listTest 1
	(integer) 1
	127.0.0.1:6379> sadd listTest 2
	(integer) 1
	127.0.0.1:6379> sadd listTest 3
	(integer) 1
	127.0.0.1:6379> smembers listTest
	1) "1"
	2) "2"
	3) "3"
	




