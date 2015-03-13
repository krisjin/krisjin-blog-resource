title: Netty WebSocket协议实例
date: 2015-03-13 23:26:36
categories: netty
tags: [netty,websocket]
---

长期以来存在着各种技术让服务器得知有新数据可用是，立即将数据发送到客户端。这些技术种类繁多，例如“推送”活comet。最常用的的一种黑客手段是对服务器发起链接创建假象，被称为长轮询。利用长轮询，客户端可以打开指向服务器的HTTP连接，而服务器会一直保持连接打开，直到发送响应。<!--more-->  


WebSocket Server代码清单：
	
	import io.netty.bootstrap.ServerBootstrap;
	import io.netty.channel.Channel;
	import io.netty.channel.ChannelInitializer;
	import io.netty.channel.ChannelPipeline;
	import io.netty.channel.EventLoopGroup;
	import io.netty.channel.nio.NioEventLoopGroup;
	import io.netty.channel.socket.SocketChannel;
	import io.netty.channel.socket.nio.NioServerSocketChannel;
	import io.netty.handler.codec.http.HttpObjectAggregator;
	import io.netty.handler.codec.http.HttpServerCodec;
	import io.netty.handler.stream.ChunkedWriteHandler;
	
	/**
	 * @author krisjin
	 */
	public class WebSocketServer {

	public void run(int port) throws InterruptedException {
		EventLoopGroup bossGroup = new NioEventLoopGroup();

		EventLoopGroup workerGroup = new NioEventLoopGroup();

		try {
			ServerBootstrap boot = new ServerBootstrap();
			boot.group(bossGroup, workerGroup);
			boot.channel(NioServerSocketChannel.class);
			boot.childHandler(new ChannelInitializer<SocketChannel>() {

				protected void initChannel(SocketChannel ch) throws Exception {
					ChannelPipeline pipeline = ch.pipeline();
					pipeline.addLast("http-codec", new HttpServerCodec());
					pipeline.addLast("aggregator", new HttpObjectAggregator(65536));
					pipeline.addLast("http-chunked", new ChunkedWriteHandler());
					pipeline.addLast("handler", new WebSocketServerHandler());
				}

			});
			Channel ch = boot.bind(port).sync().channel();
			System.out.println("Web socket server started at port " + port + '.');
			System.out.println("Open your browser to http://localhost:" + port + '/');
			ch.closeFuture().sync();
			
		} finally {
			bossGroup.shutdownGracefully();
			workerGroup.shutdownGracefully();
		}

	}

	public static void main(String[] args) throws InterruptedException {
		
		int port = 8080;
		if (args.length > 0) {
		    try {
			port = Integer.parseInt(args[0]);
		    } catch (NumberFormatException e) {
			e.printStackTrace();
		    }
		}
		new WebSocketServer().run(port);

		}
	}


handler代码清单：

	import io.netty.buffer.ByteBuf;
	import io.netty.buffer.Unpooled;
	import io.netty.channel.ChannelFuture;
	import io.netty.channel.ChannelFutureListener;
	import io.netty.channel.ChannelHandlerContext;
	import io.netty.channel.SimpleChannelInboundHandler;
	import io.netty.handler.codec.http.DefaultFullHttpResponse;
	import io.netty.handler.codec.http.FullHttpRequest;
	import io.netty.handler.codec.http.FullHttpResponse;
	import io.netty.handler.codec.http.HttpHeaders;
	import io.netty.handler.codec.http.HttpResponseStatus;
	import io.netty.handler.codec.http.HttpVersion;
	import io.netty.handler.codec.http.websocketx.CloseWebSocketFrame;
	import io.netty.handler.codec.http.websocketx.PingWebSocketFrame;
	import io.netty.handler.codec.http.websocketx.PongWebSocketFrame;
	import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
	import io.netty.handler.codec.http.websocketx.WebSocketFrame;
	import io.netty.handler.codec.http.websocketx.WebSocketServerHandshaker;
	import io.netty.handler.codec.http.websocketx.WebSocketServerHandshakerFactory;
	import io.netty.util.CharsetUtil;
	
	import java.util.Date;
	
	/**
	 * @author krisjin
	 */
	public class WebSocketServerHandler extends SimpleChannelInboundHandler<Object> {

	private WebSocketServerHandshaker handshaker;

	protected void messageReceived(ChannelHandlerContext ctx, Object msg) throws Exception {

		if (msg instanceof FullHttpRequest) {
			handleHttpRequest(ctx, (FullHttpRequest) msg);
		} else if (msg instanceof WebSocketFrame) {
			handleWebSocketFrame(ctx, (WebSocketFrame) msg);
		}
	}

	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		ctx.flush();
	}

	public void handleHttpRequest(ChannelHandlerContext ctx, FullHttpRequest req) {
		if (!req.getDecoderResult().isSuccess() || !"websocket".equals(req.headers().get("Upgrade"))) {
			sendHttpResponse(ctx, req, new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.BAD_REQUEST));
			return;
		}

		// 构造握手响应返回，本机测试
		WebSocketServerHandshakerFactory wsFactory = new WebSocketServerHandshakerFactory("ws://localhost:8080/websocket", null,
				false);

		handshaker = wsFactory.newHandshaker(req);

		if (handshaker == null) {
			WebSocketServerHandshakerFactory.sendUnsupportedWebSocketVersionResponse(ctx.channel());
		} else {
			handshaker.handshake(ctx.channel(), req);
		}
	}

	public void handleWebSocketFrame(ChannelHandlerContext ctx, WebSocketFrame frame) {

		// 判断是否是关闭链路的指令
		if (frame instanceof CloseWebSocketFrame) {
			handshaker.close(ctx.channel(), (CloseWebSocketFrame) frame.retain());
			return;
		}
		// 判断是否是Ping消息
		if (frame instanceof PingWebSocketFrame) {
			ctx.channel().write(new PongWebSocketFrame(frame.content().retain()));
			return;
		}

		if (!(frame instanceof TextWebSocketFrame)) {
			throw new UnsupportedOperationException(String.format("%s frame types not supported", frame.getClass().getName()));
		}
		//返回应答消息
		String request= ((TextWebSocketFrame)frame).text();
		System.out.println(String.format("%s received %s", ctx.channel(), request));
		
		ctx.channel().write(new TextWebSocketFrame(request+" ,现在时刻："+new Date()));
		
	}

	private void sendHttpResponse(ChannelHandlerContext ctx, FullHttpRequest req, FullHttpResponse resp) {
		if (resp.getStatus().code() != 200) {
			ByteBuf buf = Unpooled.copiedBuffer(resp.getStatus().toString(), CharsetUtil.UTF_8);
			resp.content().writeBytes(buf);
			buf.release();
			HttpHeaders.setContentLength(resp, req.content().readableBytes());
		}
		ChannelFuture f = ctx.channel().writeAndFlush(resp);
		if (!HttpHeaders.isKeepAlive(req) || resp.getStatus().code() != 200) {
			f.addListener(ChannelFutureListener.CLOSE);
		}
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		cause.printStackTrace();
		ctx.close();
	}

	}


html客户端代码清单：
	
	<!DOCTYPE html>
	<html>
	<head>
	<meta charset="UTF-8">
	Netty WebSocket 时间服务器
	</head>
	<br>
	<body>
	<br>
	<script type="text/javascript">
	var socket;
	if (!window.WebSocket) 
	{
		window.WebSocket = window.MozWebSocket;
	}
	if (window.WebSocket) {
		socket = new WebSocket("ws://localhost:8080/websocket");
		socket.onmessage = function(event) {
			var ta = document.getElementById('responseText');
			ta.value="";
			ta.value = event.data
		};
		socket.onopen = function(event) {
			var ta = document.getElementById('responseText');
			ta.value = "打开WebSocket服务正常，浏览器支持WebSocket!";
		};
		socket.onclose = function(event) {
			var ta = document.getElementById('responseText');
			ta.value = "";
			ta.value = "WebSocket 关闭!"; 
		};
	}
	else
		{
		alert("抱歉，您的浏览器不支持WebSocket协议!");
		}
	
	function send(message) {
		if (!window.WebSocket) { return; }
		if (socket.readyState == WebSocket.OPEN) {
			socket.send(message);
		}
		else
			{
			  alert("WebSocket连接没有建立成功!");
			}
	}
	</script>
	<form onsubmit="return false;">
	<input type="text" name="message" value="Netty最佳实践"/>
	<br><br>
	<input type="button" value="发送WebSocket请求消息" onclick="send(this.form.message.value)"/>
	<hr color="blue"/>
	<h3>服务端返回的应答消息</h3>
	<textarea id="responseText" style="width:500px;height:300px;"></textarea>
	</form>
	</body>
	</html>