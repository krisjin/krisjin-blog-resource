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


## 构建pipeline
事实上用户不需要创建pipeline，因为使用ServerBootstrap或Bootstrap启动服务端或客户端时，Netty会为每个Channel创建一个独立的pipeline,对于使用者而言，只需要将自定义的拦截器加入到pipeline即可，相关代码入下：

	pipeline=ch.pipeline();
	pipeline.addLast("decoder",new MyProtocolDecoder());
	pipeline.addLast("encoder",new MyProtocolEncoder());

对于编写这样的ChannelHandler，它存在先后顺序，例如 MessageToMessageDecoder,在它之前往往需要有ByteToMessageDecoder将ByteBuf编码为对象，然后对对象做二次编码得到最终的POJO对象。Pipeline支持指定位置添加或者删除拦截器，相关接口定义如下：

![](/img/channelpipeline-sort.png)

## ChannelPipeline的主要特性
ChannelPipeline支持运行态动态的添加或者删除ChannelHandler，在某些场景下这个特性非常实用。例如当业务高峰期需要对系统做拥塞保护时，就可以根据当前的系统时间进行判断，如果处于业务高峰期，则动态地将系统拥塞保护添加到当前的ChannelPipeline中，当高峰期过去之后，就可以动态的删除拥塞保护ChannelHandler了。

ChannelPipeline是线程安全的，这意味着N个业务线程可以并发地操作ChannelPipeline而不存在多线程并发问题。但是，ChannelHandler却不是线程安全的，这意味着尽管ChannelPipeline是线程安全的，但是用户仍然需要自己保证ChannelHandler的线程安全。
