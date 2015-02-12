title: netty定长消息解码
date: 2015-02-12 11:24:00
categories: netty
tags: [netty]
---

### FixedLengthFrameDecoder使用
FixedLengthFrameDecoder是netty提供的固定长度解码器，它能够按照指定的长度对消息进行解码，开发者不需要考虑TCP的粘包拆包问题，非常使用。下面通过一个实例来具体了解一下：

1、**服务端:**  
在服务端的ChannelPipeline中新增FixedLengthFrameDecoder，消息截取长度自定义。
<!--more-->

	public class EchoServer {

	public void bind(int port) throws InterruptedException {

		EventLoopGroup bossGroup = new NioEventLoopGroup();

		EventLoopGroup workerGroup = new NioEventLoopGroup();

		try {

			ServerBootstrap boot = new ServerBootstrap();

			boot.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
					.option(ChannelOption.SO_BACKLOG, 100).childHandler(new ChannelInitializer<SocketChannel>() {

						@Override
						protected void initChannel(SocketChannel ch) throws Exception {
							ch.pipeline().addLast(new FixedLengthFrameDecoder(7));
							ch.pipeline().addLast(new StringDecoder());
							ch.pipeline().addLast(new EchoServerHandler());
						}
					});

			// 绑定端口，同步等待成功
			ChannelFuture cf = boot.bind(port).sync();
			// 等待服务器监听端口关闭
			cf.channel().closeFuture().sync();
		} finally {
			bossGroup.shutdownGracefully();
			workerGroup.shutdownGracefully();
		}
	}

	public static void main(String[] args) {
		int port = 8080;
		try {
			new EchoServer().bind(port);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	}

**服务端handler**

	public class EchoServerHandler extends ChannelHandlerAdapter {

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		cause.printStackTrace();
		ctx.close();
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		System.out.println("Receive client : [" + msg + "]");

	}

	}

2、**客户端** 

	public class EchoClient {

	public void connect(int port, String host) throws InterruptedException {

		EventLoopGroup group = new NioEventLoopGroup();

		try {
			Bootstrap boot = new Bootstrap();

			boot.group(group).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true);

			boot.handler(new ChannelInitializer<SocketChannel>() {
				@Override
				protected void initChannel(SocketChannel ch) throws Exception {
					ch.pipeline().addLast(new EchoClientHandler());
				}

			});

			ChannelFuture cf = boot.connect(host, port).sync();
			cf.channel().closeFuture().sync();

		} finally {
			group.shutdownGracefully();
		}
	}

	public static void main(String[] args) throws InterruptedException {
		new EchoClient().connect(8080, "127.0.0.1");
	}

	}


**客户端handler**
	public class EchoClientHandler extends ChannelHandlerAdapter {

	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		super.channelRead(ctx, msg);
	}

	public void channelActive(ChannelHandlerContext ctx) throws Exception {

		ByteBuf out = Unpooled.copiedBuffer("Krisjin  Welcome to Beijing!".getBytes());
		ctx.writeAndFlush(out);

	}

	}

