title: Netty包功能描述
date: 2015-02-16 12:24:42
categories: netty
tags:
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在学习Netty的过程中为了从整体的角度去了解整个框架的功能实现，一种可以去官网看用户指南，另一种是查看API，当然我相信还有很多种，还有哪些学习方式，可以留言哦。  


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了了解netty的子项目都用于那些功能实现，下面是对API中包的功能描述翻译，针对的版本是5.0，有不当之处，请多多提意见留言。

<!--more-->

#### <span style="color:blue;">底层数据表示</span>

1. **io.netty.buffer**  
抽象的字节缓冲区：使用基本的数据结构来表示一个二进制信息和文本信息


#### <span style="color:blue;">**所有IO操作的重要接口**</span>

1. **io.netty.channel**  
核心Channel API:它是异步的、基于事件驱动的抽象的各种传送接口

2. **io.netty.channel.embedded**  
虚拟通道是帮助包装了一系列的处理程序进行单元测试的处理程序，或使用他们在非I/ O方面

3. **io.netty.channel.group**  
一个通道注册，帮助用户保持Channel列表打开和并对它们进行执行多个操作 

4. **io.netty.channel.local**  
一种虚拟传送方式，允许同一个虚拟机上的两个部分可以相互通信

5. **io.netty.channel.nio**   
基于NIO的Channel实现，推荐用于大量连接的（>=1000）

6. **io.netty.channel.oio**  
基于老IO的Channel实现，推荐用于小数量的连接（<1000）

7. **io.netty.channel.rxtx**  
基于RXTX串行和并行端口通信传输

8. **io.netty.channel.sctp**  
继承核心渠道API的抽象SCTP套接字接口

9. **io.netty.channel.sctp.nio**  
基于NIO的SCTP Channel 实现 ，推荐了大量的连接（>=1000）

10. **io.netty.channel.sctp.oio**  
基于SCTP Channel实现，使用旧的阻塞式IO，推荐一个小数量的连接（<1000）

11. **io.netty.channel.socket**
继承了核心Channel接口的抽象TCP和UDP套接字接口 

12. **io.netty.channel.socket.nio**  
基于NIO的Socket Channel实现，推荐了大量的连接（>=1000）

13. **io.netty.channel.socket.oio**  
基于旧IO的Socket Channel实现，推荐一个小数量的连接（<1000）

14. **io.netty.channel.udt**  
udt传输协议

15. **io.netty.channel.udt.nio**  
基于UDT协议传输方式实现的NIOChannel

#### <span style="color:blue;">客户端和服务器引导操作</span>

1. **io.netty.bootstrap**  
使用流畅的帮助类很容易的实现客户端与服务端的channel初始化操作


#### <span style="color:blue;">可重复使用的I / O事件拦截器</span>

1. **io.netty.handler.codec**  
通用的可扩展的解码器实现，如如TCP/ IP中的数据包分段和重组问题

2. **io.netty.handler.codec.base64**  
编码器和解码器：转换Base64编码成String或者ByteBuf解码成ByteBuf,反之亦然

3. **io.netty.handler.codec.bytes**  
编码器和解码器转换字节数组到ByteBuf，反之亦然  

4. **io.netty.handler.codec.compression**  
编码器和解码器，压缩和解压缩ByteBufs的压缩格式如zlib，gzip和Snappy

5. **io.netty.handler.codec.http**  
编码器，解码器和HTTP及其相关消息类型

6. **io.netty.handler.codec.http.cors**  
该软件包包含跨起源资源共享（CORS）相关的类

7. **io.netty.handler.codec.http.multipart**  
HTTP多部分支持 

8. **io.netty.handler.codec.http.websocketx**  
编码器，解码器，handshakers 和它们websocket数据帧的相关数据类型

9. **io.netty.handler.codec.marshalling**  
使用JBoss的Marshalling实现编码器、解码器


10. **io.netty.handler.codec.memcache**  
ASCII和二进制类常见的超集

11. **io.netty.handler.codec.memcache.binary**  
Memcache二进制协议的接口和实现

12. **io.netty.handler.codec.protobuf**  
编码器、解码器，使用Google的 protobuf 转换成ByteBuf，反之亦然。  


13. **io.netty.handler.codec.rtsp**  
基于HTTP的编解码器的RTSP扩展

14. **io.netty.handler.codec.sctp**  
解码器和编码器来管理信息，并完成多流编解码器在SCTP/ IP

15. **io.netty.handler.codec.serialization**  
编码器，解码器和其兼容性流实现它把一个序列化对象转成字节缓冲区，反之亦然

16. **io.netty.handler.codec.socks**  
socks相关消息类型的编码器、解码器

17. **io.netty.handler.codec.spdy**  
SPDY协议的编码器、解码器、会话处理和它们的相关消息类型

18. **io.netty.handler.codec.string**  
编码器和解码器，把一个字符串变成ByteBuf，反之亦然

19. **io.netty.handler.codec.xml**  
XML特定的编解码器

20. **io.netty.handler.logging**  
记录用于调试目的I / O事件

21. **io.netty.handler.ssl**  
SSL·基于TLS SSLEngine的实施

22. **io.netty.handler.stream**  
写非常大的数据流异步既不花费了大量的内存，也没有得到OutOfMemoryError

23. **io.netty.handler.timeout** 
增加了对读取和写入超时并使用定时器空闲连接通知的支持

24. **io.netty.handler.traffic**  
传输成型处理和动态统计的实现

#### <span style="color:blue;">其它</span>

1. **io.netty.util**  
使用工具类

2. **io.netty.util.concurrent**  
并发/异步任务工具类


