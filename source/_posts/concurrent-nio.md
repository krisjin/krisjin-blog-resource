title: 高吞吐高并发Java NIO服务的架构
date: 2015-04-08 22:45:39
categories: concurrent
tags:
---


Java NIO成功的应用在了各种分布式、即时通信和中间件Java系统中。证明了基于NIO构建的通信基础，是一种高效，且扩展性很强的通信架构。<!--more-->

基于Reactor模式的高可扩展性架构这个架构的基本思路在“基于高可用性NIO服务器架构”（http://today.java.net/pub/a/today/2007/02/13/architecture-of-highly-scalable-nio-server.html
）中有了清晰的论述。经过几年实际运营的经验，这种架构的灵活性得到了很好的验证。我们注意几点，

1. 一个小的线程池负责dispatch NIO事件。
2. 注册事件，即操作selecter时，要使用一个同步锁（即Architecture of a Highly Scalable NIO-Based Server一文中的guard对象），即对同一个selector的操作是互斥的。
3. 这个小的线程池不处理逻辑业务，大小可以是Runtime.getRuntime().availableProcessors() + 1，即你系统有效CPU个数+1。这是因为我们假设有一个线程专门处理accept事件，
而其他线程处理read/write操作。
4. 用另一个单独的线程池处理逻辑业务

那么基于NIO实现高效和高可扩展服务，还有哪些构架方面的问题需要考虑呢？
NIO构架中比较需要经验和比较复杂的主要是2点：1,）是基于提高的性能的线程池设计；2）基于网络通讯量的通讯完整性校验的构架。

#### 1. 基于提高的性能的线程池设计
既然有一个单独处理逻辑业务的线程池，这个线程池的大小应该由你的业务来决定。对于高效服务器来说，这个线程池大小会对你的服务性能产生很大的影响。设置多少合适呢？

这里真的有很多情况需要考虑，换句话说，这里水很深。我只能根据自己的经验举几个例子。真正到了运营系统上，一边测试一边调整一边总结吧。

假设消息解析用时5毫秒，数据库操作用时20毫秒，其他逻辑处理用时20毫秒，那么整个业务处理用时45毫秒。
因为数据库操作主要是IO读写操作，为使CPU得到最大程度的利用，在一个16核的服务器上，应该设置 （45/ 25)
 * 16 = 29 个线程即可。

假设不是所有的操作都是在平均时间内完成，比如数据库操作，假设是在12~35毫秒区间内。即有线程会不断的被某些操作block住，为了充分利用CPU能力，因设置为（（35 + 25）/ 25）* 16 = 39个线程。

所以原则上，如果应用是一个偏重数据库操作的应用，则线程数应高些；如果应用是一个高CPU应用，则线程数不用太高。

假设逻辑处理中，对共享资源的操作用时5毫秒。此时同时只能有一个线程对共享资源进行操作，那么在一个16核的服务器上，应该设置 (37 / 5) * 1 = 8 个线程即可。

假设只有一部分操作对共享资源有写，其他只是读。这样采用乐观锁，使写操作降为所有操作的10%，那么有90%的业务，其合适的线程数可为39个线程。10%的业务应为8个线程。平均则为 35 + 1 = 36个线程。可见仔细的分析共享资源的使用，能很好的提高系统性能。

#### 2. 基于网络通讯量的通讯完整性校验的构架

先看看READ事件的触发条件：
If the selector detects that the corresponding channel is ready for reading, has reached end-of-stream, has been remotely shut down for further reading, 
or has an error pending, then it will add OP_READ to the key's ready-operation set and add the key to its selected-key set.

就是说，NIO构架中不能保证每次READ事件发生时从channel中读出的数据就是完整。例如，在通讯数据量较大时，网络层write buffer很容易被写满。此时读到的数据就是不完整的。
从构架角度，应根据应用场景设计三种不同的处理方式。

基本上有三种类型的应用，

1. 较低的通信量应用。这类应用的特点是所有的通信量不是很大，而且数据包小。所有数据都能在一次网络层buffer flush中全部写出。比如ZooKeeper client对cluster的操作。这种通信模式是完全不需要进行数据包校验的。

2. 基于RPC模式的应用。比如Hadoop，每次NameNode和DataNode之间的通讯都是通过RPC框架封装，转变成client对server的调用。所有的操作都是通过Java反射机制反射成方法调用，这样操作的特点是每次读到的数据都是可以通过ObjectInputStream(new ByteArrayInputStream(bytes)).readObject()操作的。这样的应用，应该在第一种应用的架构基础上增加对ObjectInputStream的校验。如果校验失败，则说明这次通信没有完成，应和下次read到数据合并在一起处理。

3. 基于大量数据通信的应用。这种应用的特点是基于一种大数据量通信协议，比如RTSP。数据包是否完整需要经过通信协议约定的校验符进行校验。这样就必须实现一个校验类。如果校验失败，则说明这次通信没有完成，应和下次read到数据合并在一起处理。


转自：http://blog.sina.com.cn/s/blog_6ae994a30100tcwd.html
