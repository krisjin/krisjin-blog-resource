title: netty入门实例
date: 2015-02-06 16:55:50
categories: netty
tags: [netty]
---

## netty介绍

Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server.

'Quick and easy' doesn't mean that a resulting application will suffer from a maintainability or a performance issue. Netty has been designed carefully with the experiences earned from the implementation of a lot of protocols such as FTP, SMTP, HTTP, and various binary and text-based legacy protocols. As a result, Netty has succeeded to find a way to achieve ease of development, performance, stability, and flexibility without a compromise.

<!--more-->

## 服务端

**服务端代码：**

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

**服务端handler**

    public class TimeServerHandler extends ChannelHandlerAdapter {

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		ByteBuf buf = (ByteBuf) msg;

		byte[] req = new byte[buf.readableBytes()];

		buf.readBytes(req);
		
		String body = new String(req, "UTF-8");
		
		System.out.println("The time server receive order :" + body);

		String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body) ? new java.util.Date(System.currentTimeMillis())
				.toString() : "BAD ORDER";

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


## 客户端

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
	private final ByteBuf firstMessage;

	public TimeClientHandler() {
		byte[] req = "QUERY TIME ORDER".getBytes();
		firstMessage = Unpooled.buffer(req.length);
		firstMessage.writeBytes(req);
	}

	public void channelActive(ChannelHandlerContext ctx) throws Exception {

		ctx.writeAndFlush(firstMessage);

	}

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		ByteBuf buf = (ByteBuf) msg;

		byte[] req = new byte[buf.readableBytes()];
		buf.readBytes(req);
		String body = new String(req, "UTF-8");

		System.out.println("Now is :" + body);

	}

	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {

		ctx.close();

	}

	}