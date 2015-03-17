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

**触发inbound事件**的方法如下：  

1. public ChannelPipeline fireChannelRegistered()：Channel注册事件
2. public ChannelHandlerContext fireChannelActive()：TCP链路建立成功，Channel激活事件
3. public ChannelHandlerContext fireChannelRead(Object msg)：读事件
4. public ChannelHandlerContext fireChannelReadComplete() ：读操作完成通知事件
5. public ChannelHandlerContext fireExceptionCaught(Throwable cause)：异常通知事件
6. public ChannelHandlerContext fireUserEventTriggered(Object event)：用户自定义事件
7. public ChannelHandlerContext fireChannelWritabilityChanged()：Channel的可写状态变化通知事件
8. public ChannelHandlerContext fireChannelInactive() ：TCP连接关闭，链路不可用通知事件

outbound事件通常是由用户主动发起的网络I/O操作，例如用户发起的连接操作、绑定操作、消息发送等操作，它对应上图的右半部分。

**触发outbound事件**的方法如下：

1. public ChannelFuture bind(final SocketAddress localAddress, final ChannelPromise promise)：绑定本地地址事件
2. **public ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise)**：连接服务端事件
3. **public ChannelFuture write(Object msg, ChannelPromise promise)**：发送事件
4. **public ChannelHandlerContext flush()**：刷新事件
5. **public ChannelHandlerContext read()**：读事件
6. **public ChannelFuture disconnect(ChannelPromise promise)**：断开连接事件
7. **public ChannelFuture close(ChannelPromise promise)**：关闭当前Channel事件



## 自定义拦截器
---
ChannelPipeline通过ChannelHandler接口来实现事件的拦截和处理，由于ChannelHandler中的事件种类繁多，不同的ChannelHandler可能只需要关心其中的某一个或者几个事件，所以，通常ChannelHandler只需要继承ChannelHandlerAdapter类覆盖自己关心的方法即可。
例如下面的例子展示拦截Channel Active事件，打印TCP链路建立成功日志，代码如下：

	public class ChannelActiveInbound extends ChannelHandlerAdapter {

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		
		System.out.println("TCP Connected!");
		
		ctx.fireChannelActive();
	}

	}


下面的例子展示了如何在链路关闭的时候释放资源，示例代码如下：

	public class ChannelCloseOutbound extends ChannelHandlerAdapter {

	@Override
	public void close(ChannelHandlerContext ctx, ChannelPromise promise)
			throws Exception {

		System.out.println("TCP Closing!");
		ctx.close(promise);
	}

	}
