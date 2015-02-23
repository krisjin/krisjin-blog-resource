title: netty文件传输
date: 2015-02-23 18:37:03
categories: netty
tags:
---

文件传输通常采用FTP或者HTTP附件的方式，事实上通过TCP Socket+File的方式进行传输也有一定的应用场景，尽管不是主流，但是掌握文件传输的方式还是很重要的，特别是针对两个跨主机的JVM进程之间进行持久化数据的相互交换。<!--more-->

下面使用netty实现文件传输：

**文件服务器**

	public class FileServer {

	public void run(int port) throws InterruptedException {

		EventLoopGroup bossGroup = new NioEventLoopGroup();
		EventLoopGroup workerGroup = new NioEventLoopGroup();

		try {
			ServerBootstrap boot = new ServerBootstrap();
			boot.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class).option(ChannelOption.SO_BACKLOG, 100);
			boot.childHandler(new ChannelInitializer<SocketChannel>() {

				@Override
				protected void initChannel(SocketChannel ch) throws Exception {
					ch.pipeline().addLast(new StringEncoder(CharsetUtil.UTF_8));
					ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
					ch.pipeline().addLast(new StringDecoder(CharsetUtil.UTF_8));
					ch.pipeline().addLast(new FileServerHandler());
				}

			});
			ChannelFuture cf = boot.bind(port).sync();
			cf.channel().closeFuture().sync();

		} finally {
			bossGroup.shutdownGracefully();
			workerGroup.shutdownGracefully();
		}
	}

	public static void main(String[] args) throws InterruptedException {

		new FileServer().run(8080);

	}
	}

**文件服务器端处理类**

	public class FileServerHandler extends SimpleChannelInboundHandler<String> {

	private static final String CR = System.getProperty("line.separator");

	@Override
	protected void messageReceived(ChannelHandlerContext ctx, String msg) throws Exception {

		File file = new File(msg);
		if (file.exists()) {
			if (!file.isFile()) {
				ctx.writeAndFlush("not a file :" + file + CR);
				return;
			}
			ctx.write(file + " " + file.length() + CR);

			RandomAccessFile raf = new RandomAccessFile(file, "r");
			FileRegion fileRegion = new DefaultFileRegion(raf.getChannel(), 0, raf.length());

			ctx.write(fileRegion);
			ctx.writeAndFlush(CR);
			raf.close();

		} else {
			ctx.writeAndFlush("file not found: " + file + CR);
		}
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		cause.printStackTrace();
		ctx.close();
	}

	}