title: netty分隔符解码
date: 2015-02-11 10:17:35
categories: netty
tags: [netty]
---
### DelimiterBasedFrameDecoder分隔符解码

上一章介绍了换行的粘包拆包处理，使用到的类是LineBasedFrameDecoder和StringDecoder
这章介绍分隔符解码的处理。

<!--more-->


**服务端代码**

	public class EchoServer {

	public void bind(int port) throws Exception {

		// 配置服务端线程组
		EventLoopGroup bossGroup = new NioEventLoopGroup();
		EventLoopGroup workerGroup = new NioEventLoopGroup();

		ServerBootstrap boot = new ServerBootstrap();
		try {
			boot.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
					.option(ChannelOption.SO_BACKLOG, 100).handler(new LoggingHandler(LogLevel.INFO))
					.childHandler(new ChannelInitializer<SocketChannel>() {

						@Override
						protected void initChannel(SocketChannel ch) throws Exception {

							ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
							ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiter));
							ch.pipeline().addLast(new StringDecoder());
							ch.pipeline().addLast(new EchoServerHandler());
						}

					});

			// 绑定端口，同步等待成功
			ChannelFuture cf = boot.bind(port).sync();
			// 等待服务器端监听端口关闭
			cf.channel().closeFuture().sync();
		} finally {
			// 优雅退出，释放线程池资源
			bossGroup.shutdownGracefully();
			workerGroup.shutdownGracefully();
		}
	}

	public static void main(String[] args) throws Exception {
		int port = 8080;
		if (args.length > 0) {
			port = Integer.valueOf(args[0]);
		}

		new EchoServer().bind(port);
	}
	}


**服务端handler**

	public class EchoServerHandler extends ChannelHandlerAdapter {

	int counter = 0;

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		String body = (String) msg;
		System.out.println("This is " + ++counter + " times receive client : [" + body + "]");

		body += "$_";
		ByteBuf echo =Unpooled.copiedBuffer(body.getBytes());
		
		ctx.writeAndFlush(echo);
		
	}

	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		cause.printStackTrace();
		ctx.close();
	}
	}


**客户端代码**
	
	public class EchoClient {

	public void connect(int port, String host) throws Exception {

		EventLoopGroup group = new NioEventLoopGroup();

		try {
			Bootstrap boot = new Bootstrap();
			boot.group(group).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
					.handler(new ChannelInitializer<SocketChannel>() {

						@Override
						protected void initChannel(SocketChannel ch) throws Exception {

							ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
							ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiter));
							ch.pipeline().addLast(new StringDecoder());
							ch.pipeline().addLast(new EchoClientHandler());

						}

					});

			// 发起异步连接操作
			ChannelFuture cf = boot.connect(host, port).sync();
			// 等待客户端链路关闭
			cf.channel().closeFuture().sync();

		} finally {
			// 优雅退出，释放NIO线程组
			group.shutdownGracefully();
		}

	}

	public static void main(String[] args) throws Exception {
		int port = 8080;
		String host = "127.0.0.1";
		new EchoClient().connect(port, host);
	}

	}


**客户端handler**

	public class EchoClientHandler extends ChannelHandlerAdapter {
	int counter = 0;
	static final String ECHO_REQ = "Hi, krisjin. Welcome to Netty.$_";

	public EchoClientHandler() {

	}

	public void channelActive(ChannelHandlerContext ctx) {
		for (int i = 0; i < 10; i++) {
			ByteBuf write = Unpooled.copiedBuffer(ECHO_REQ.getBytes());

			ctx.writeAndFlush(write);

		}
	}

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		System.out.println("This is " + ++counter + "times recevie server : [" + msg + "]");

	}

	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		ctx.flush();
	}

	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
		cause.printStackTrace();
		ctx.close();
	}
	}
