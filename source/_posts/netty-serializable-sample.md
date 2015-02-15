title: netty序列化实现
date: 2015-02-13 13:35:46
categories: netty
tags: [netty]
---

### Java序列化
jdk本身提供了序列化的实现，需要序列化的类要实现Serializable接口。
序列化的目的主要有网络传输和对象持久化，网络传输应用到远程系统之间的调用。持久化在ORM
中使用的较多。java序列化不支持多种语言的实现，假如A系统是用java 序列化请求到B系统，而B系统是用C++实现，在这种情况下Java序列化并不是很适用，很多成熟的远程调用框架也没有采用java 自带的序列化,java的序列化性能也很低
<!--more-->
### netty序列化
使用netty提供的序列化，序列化类也同样要实现Serializable接口，提高了两个序列化类是ObjectDecoder，ObjectEncoder，下面通过具体实例来了解下：

**服务器端**

	public class OrderServer {

	public void bind(int port) throws InterruptedException {
		EventLoopGroup bossGroup = new NioEventLoopGroup();
		EventLoopGroup workerGroup = new NioEventLoopGroup();

		try {
			ServerBootstrap boot = new ServerBootstrap();
			boot.group(bossGroup, workerGroup)
				.channel(NioServerSocketChannel.class)
				.option(ChannelOption.SO_BACKLOG, 100).handler(new LoggingHandler(LogLevel.INFO))
				.childHandler(new ChannelInitializer<SocketChannel>() {

						@Override
						protected void initChannel(SocketChannel ch) throws Exception {
							ch.pipeline().addLast(new ObjectDecoder(1024 * 1024, ClassResolvers.weakCachingConcurrentResolver(this.getClass().getClassLoader())));
							ch.pipeline().addLast(new ObjectEncoder());
							ch.pipeline().addLast(new OrderServerHandler());

						}
					});

			
			//绑定端口，同步等待
			ChannelFuture cf =boot.bind(port).sync();
			//等待服务端监听端口关闭
			cf.channel().closeFuture().sync();
			
		}  finally {
			bossGroup.shutdownGracefully();
			workerGroup.shutdownGracefully();
		}

	}

	public static void main(String[] args) throws InterruptedException {
		new OrderServer().bind(8080);
	}

	}


**服务器端handler**
  
	public class OrderServerHandler extends ChannelHandlerAdapter {


	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		OrderRequest request = (OrderRequest) msg;

		if ("krisjin".equals(request.getUserName())) {

			System.out.println("Service accept client order request: [" + request.toString() + "]");
			ctx.writeAndFlush(getResponse(request.getOrderId()));
		}

	}

	private OrderResponse getResponse(long orderId) {
		OrderResponse resp = new OrderResponse();

		resp.setOrderId(orderId);
		resp.setRespCode(0);
		resp.setDesc("Receive order success,we are as quickly as possible send...");
		return resp;
	}
	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		cause.printStackTrace();
		ctx.close();
	}
	}	

**客户端**

	public class OrderClient {

	public void connect(int port, String host) {
		EventLoopGroup group = new NioEventLoopGroup();

		try {
			Bootstrap boot = new Bootstrap();
			boot.group(group).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
					.handler(new ChannelInitializer<SocketChannel>() {

						@Override
						protected void initChannel(SocketChannel ch) throws Exception {

							ch.pipeline().addLast(
									new ObjectDecoder(1024, ClassResolvers.cacheDisabled(this.getClass()
											.getClassLoader())));
							ch.pipeline().addLast(new ObjectEncoder());
							ch.pipeline().addLast(new OrderClientHandler());
						}
					});

			// 发起异步连接操作
			ChannelFuture cf = boot.connect(host, port).sync();
			cf.channel().closeFuture().sync();

		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			group.shutdownGracefully();
		}

	}

	public static void main(String[] args) {
		new OrderClient().connect(8080, "127.0.0.1");
	}
	}

**客户端handler**

	public class OrderClientHandler extends ChannelHandlerAdapter {

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {

		for (int i = 1; i <= 10; i++) {
			ctx.write(buildRequest(i));
		}
		ctx.flush();
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

		System.out.println("Receive server response :" + msg);

	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		cause.printStackTrace();
		ctx.close();
	}

	private OrderRequest buildRequest(long orderId) {
		OrderRequest request = new OrderRequest();
		request.setOrderId(orderId);
		request.setUserName("krisjin");
		request.setAddress("朝阳门外大街泛利大厦");
		request.setMobilePhone("10000000");
		request.setProductName("黑客与画家");
		return request;
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		ctx.flush();
	}

	}
