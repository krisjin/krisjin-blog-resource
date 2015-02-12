title: tcp粘包拆包实例
date: 2015-02-09 15:23:20
categories: netty
tags: [netty]
---
### 1.TCP粘包/拆包
TCP是个"流"协议，所谓流，就是没有界限的一串数据。大家可以想想河里的水，是连成一片的，其间并没有分界线。TCP底层并不了解上层业务数据的具体含义，它会根据TCP缓冲区的实际情况进行包的划分，所以在业务上认为，一个完整的包可能被TCP拆分成多个包进行发送，也有可能把多个小的包封装成一个大的数据包发送，这就是所谓的TCP粘包和拆包问题。
<!--more-->
### 2.TCP粘包拆包问题说明
通过下图进行说明：

![](/img/tcp-stick.png)  
假设客户端分别发送了两个数据包D1和D2给服务端，由于服务端一次读取的字节数是不确定的，所以可能存在以下四种情况。

1. 服务端分两次读取了两个独立的数据包，分别是D1和D2,没有粘包和拆包。
2. 服务端一次读取到了两个数据包，D1和D2粘合在一起，被称为TCP粘包。
3. 服务端分两次读取到了两个数据包，第一次读取了完整的D1包和D2包的部分内容，第二次读取到了D2包的剩余内容，这被称为ＴＣＰ拆包。
4. 服务端分两次读取到了两个数据包，第一次读取了D1包的部分内容D1_1,第二次读取了D1的剩余内容D1_2和D2包的整包。

如果此时tcp接收滑窗非常小，而数据包D1和D2比较大。很有可能发生第五种情况，即服务端分多次才能将D1和D2包接收完全。

### 3.TCP粘包拆包发生的原因
问题产生的原因有三个，分别如下：

1. 应用程序写入的字节大小大于套接口发送缓冲区大小。
2. 进行MSS大小的TCP分段。
3. 以太网侦测的payload大于MTU进行IP分片。
图示：
![](/img/tcp-stick-problem.png)


### 4.粘包问题的解决策略
由于底层的TCP无法理解上层的业务数据，所以在底层是无法保证数据包不被拆分和重组的，这个问题只能通过上层的应用协议栈设计来解决，根据业界的主流协议的解决方案如下：

1. 消息定长，例如每个报文的大小固定长度为200字节，如果不够，空位补空格。
2. 在包尾增加回车换行符进行分割，例如FTP协议。
3. 将消息分为消息头和消息体，消息头包含表示消息体总长度的字段，通常设计思路为消息头的第一个字段使用int32来表示消息的总长度。
4. 更复杂的应用层协议。


### 5.实例代码

**服务器端代码**
	public class TimeServer {

	public void bind(int port) throws Exception {

		EventLoopGroup bossGroup = new NioEventLoopGroup();

		EventLoopGroup workderGroup = new NioEventLoopGroup();

		ServerBootstrap b = new ServerBootstrap();
		try {
			b.group(bossGroup, workderGroup).channel(NioServerSocketChannel.class)
					.option(ChannelOption.SO_BACKLOG, 1024).childHandler(new ChildChannelHandler());

			// 绑定端口，同步等待成功
			ChannelFuture f = b.bind(port).sync();

			// 等待服务器端端口关闭
			f.channel().closeFuture().sync();

		} finally {
			bossGroup.shutdownGracefully();
			workderGroup.shutdownGracefully();
		}

	}

	private class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

		protected void initChannel(SocketChannel channel) throws Exception {
			channel.pipeline().addLast(new LineBasedFrameDecoder(1024));
			channel.pipeline().addLast(new StringDecoder());
			channel.pipeline().addLast(new TimeServerHandler());
		}

	}

	public static void main(String[] args) throws Exception {
		int port = 8888;

		if (args.length > 0) {
			try {
				port = Integer.valueOf(args[0]);

			} catch (NumberFormatException e) {
				e.printStackTrace();
			}
		}
		new TimeServer().bind(port);
	}
	}

**服务器端handler**

	public class TimeServerHandler extends ChannelHandlerAdapter {

	private int counter;

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		String body = (String) msg;

		System.out.println("The time server receive order :" + body + " ; the counter is :" + ++counter);

		String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new java.util.Date(System.currentTimeMillis())
				.toString() : "BAD ORDER";

		currentTime += System.getProperty("line.separator");

		ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
		ctx.write(resp);

	}

	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		ctx.flush();
	}

	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		ctx.close();
	}

	}

**客户端代码**

	public class TimeClient {
	public void connect(int port, String host) {

		// 配置客户端NIO线程组
		EventLoopGroup group = new NioEventLoopGroup();

		Bootstrap bootstrap = new Bootstrap();

		try {
			bootstrap.group(group).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
					.handler(new ChannelInitializer<SocketChannel>() {

						protected void initChannel(SocketChannel ch) throws Exception {
							ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
							ch.pipeline().addLast(new StringDecoder());
							ch.pipeline().addLast(new TimeClientHandler());
						}

					});
			// 发起异步连接操作
			ChannelFuture f = bootstrap.connect(host, port).sync();
			// 等待客户端链路关闭
			f.channel().closeFuture().sync();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {

			group.shutdownGracefully();
		}

	}

	public static void main(String[] args) {
		int port = 8888;
		try {
			if (args.length > 0) {
				port = Integer.valueOf(args[0]);
			}
		} catch (NumberFormatException e) {
			e.printStackTrace();
		}
		new TimeClient().connect(port, "127.0.0.1");
	}
	}

**客户端handler**

	public class TimeClientHandler extends ChannelHandlerAdapter {

	private int counter;
	private byte[] req;

	public TimeClientHandler() {
		req = ("QUERY TIME ORDER" + System.getProperty("line.separator")).getBytes();
	}

	public void channelActive(ChannelHandlerContext ctx) throws Exception {

		ByteBuf message = null;

		for (int i = 0; i < 100; i++) {
			message = Unpooled.buffer(req.length);
			message.writeBytes(req);
			ctx.writeAndFlush(message);
		}

	}

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		String body = (String) msg;

		System.out.println("Now is :" + body + " ; the counter is : " + ++counter);

	}

	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		ctx.close();
	}
	}
