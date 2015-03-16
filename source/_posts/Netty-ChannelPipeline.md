title: Netty ChannelPipeline 详解
date: 2015-03-16 21:29:29
categories: netty
tags: [netty,channelpipeline]
---

ChannelPipeline是ChannelHandler的容器，它负责ChannelHandler的管理和事件拦截与调度。<!--more-->

## ChannelPipeline的事件处理
---
下图展示了一个消息被ChannelPipeline的ChannelHandler链拦截和处理的全过程。

		                           						I/O Request
	 *                                            via {@link Channel} or
	 *                                        {@link ChannelHandlerContext}
	 *                                                      |
	 *  +---------------------------------------------------+---------------+
	 *  |                           ChannelPipeline         |               |
	 *  |                                                  \|/              |
	 *  |    +----------------------------------------------+----------+    |
	 *  |    |                   ChannelHandler  N                     |    |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |              /|\                                  |               |
	 *  |               |                                  \|/              |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |    |                   ChannelHandler N-1                    |    |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |              /|\                                  .               |
	 *  |               .                                   .               |
	 *  | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
	 *  |          [method call]                      [method call]         |
	 *  |               .                                   .               |
	 *  |               .                                  \|/              |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |    |                   ChannelHandler  2                     |    |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |              /|\                                  |               |
	 *  |               |                                  \|/              |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |    |                   ChannelHandler  1                     |    |
	 *  |    +----------+-----------------------------------+----------+    |
	 *  |              /|\                                  |               |
	 *  +---------------+-----------------------------------+---------------+
	 *                  |                                  \|/
	 *  +---------------+-----------------------------------+---------------+
	 *  |               |                                   |               |
	 *  |       [ Socket.read() ]                    [ Socket.write() ]     |
	 *  |                                                                   |
	 *  |  Netty Internal I/O Threads (Transport Implementation)            |
	 *  +-------------------------------------------------------------------+

消息的读取和发送处理全流程描述如下：

1. 底层的SocketChannel read()方法读取ByteBuf,触发ChannelRead事件，由I/O线程NioEventLoop调用ChannelPipeline的fireChannelRead(Object msg)方法，将消息（Bytebuf）传输到ChannelPipeline中；
2. 消息依次被HeadHandler、ChannelHandler1、ChannelHandler2.....TailHandler拦截和处理，在这个过程中，任何ChannelHandler都可以中断当前的流程，结束消息的传递。
3. 调用ChannelHandlerContext的write方法发送消息，消息从TailHandler开始，途径ChannelHandlerN....ChannelHandler1、HeadHandler，最终被添加到消息发送缓冲区中等待刷新和发送，在此过程中也可以中断消息的传递，例如编码失败时，就需要中断流程，构造异常的Future返回。

Netty中的事件分为inbound事件和outbound事件，inbound事件通常由I/O线程触发，例如TCP链路建立事件，链路关闭事件，读事件，异常通知事件等，它对应上图的左半部分。