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

    <span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoServer</span> </span>{

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">bind</span><span class="hljs-params">(<span class="hljs-keyword">int</span> port)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        <span class="hljs-comment">// 配置服务端线程组</span>
        EventLoopGroup bossGroup = <span class="hljs-keyword">new</span> NioEventLoopGroup();
        EventLoopGroup workerGroup = <span class="hljs-keyword">new</span> NioEventLoopGroup();

        ServerBootstrap boot = <span class="hljs-keyword">new</span> ServerBootstrap();
        <span class="hljs-keyword">try</span> {
            boot.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, <span class="hljs-number">100</span>).handler(<span class="hljs-keyword">new</span> LoggingHandler(LogLevel.INFO))
                    .childHandler(<span class="hljs-keyword">new</span> ChannelInitializer<SocketChannel>() {

                        <span class="hljs-annotation">@Override</span>
                        <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> <span class="hljs-keyword">throws</span> Exception </span>{

                            ByteBuf delimiter = Unpooled.copiedBuffer(<span class="hljs-string">"$_"</span>.getBytes());
                            ch.pipeline().addLast(<span class="hljs-keyword">new</span> DelimiterBasedFrameDecoder(<span class="hljs-number">1024</span>, delimiter));
                            ch.pipeline().addLast(<span class="hljs-keyword">new</span> StringDecoder());
                            ch.pipeline().addLast(<span class="hljs-keyword">new</span> EchoServerHandler());
                        }

                    });

            <span class="hljs-comment">// 绑定端口，同步等待成功</span>
            ChannelFuture cf = boot.bind(port).sync();
            <span class="hljs-comment">// 等待服务器端监听端口关闭</span>
            cf.channel().closeFuture().sync();
        } <span class="hljs-keyword">finally</span> {
            <span class="hljs-comment">// 优雅退出，释放线程池资源</span>
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">int</span> port = <span class="hljs-number">8080</span>;
        <span class="hljs-keyword">if</span> (args.length > <span class="hljs-number">0</span>) {
            port = Integer.valueOf(args[<span class="hljs-number">0</span>]);
        }

        <span class="hljs-keyword">new</span> EchoServer().bind(port);
    }
    }
    `</pre>

    **服务端handler**

    <pre>`<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoServerHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ChannelHandlerAdapter</span> </span>{

    <span class="hljs-keyword">int</span> counter = <span class="hljs-number">0</span>;

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        String body = (String) msg;
        System.out.println(<span class="hljs-string">"This is "</span> + ++counter + <span class="hljs-string">" times receive client : ["</span> + body + <span class="hljs-string">"]"</span>);

        body += <span class="hljs-string">"$_"</span>;
        ByteBuf echo =Unpooled.copiedBuffer(body.getBytes());

        ctx.writeAndFlush(echo);

    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">exceptionCaught</span><span class="hljs-params">(ChannelHandlerContext ctx, Throwable cause)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        cause.printStackTrace();
        ctx.close();
    }
    }
    `</pre>

    **客户端代码**

    <pre>`<span class="hljs-keyword">public</span> <span class="hljs-keyword">class</span> <span class="hljs-title">EchoClient</span> {

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">connect</span><span class="hljs-params">(<span class="hljs-keyword">int</span> port, String host)</span> throws Exception </span>{

        EventLoopGroup <span class="hljs-keyword">group</span> = <span class="hljs-keyword">new</span> NioEventLoopGroup();

        <span class="hljs-keyword">try</span> {
            Bootstrap boot = <span class="hljs-keyword">new</span> Bootstrap();
            boot.<span class="hljs-keyword">group</span>(<span class="hljs-keyword">group</span>).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, <span class="hljs-keyword">true</span>)
                    .handler(<span class="hljs-keyword">new</span> ChannelInitializer<SocketChannel>() {

                        @<span class="hljs-function">Override
                        <span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> throws Exception </span>{

                            ByteBuf delimiter = Unpooled.copiedBuffer(<span class="hljs-string">"$_"</span>.getBytes());
                            ch.pipeline().addLast(<span class="hljs-keyword">new</span> DelimiterBasedFrameDecoder(<span class="hljs-number">1024</span>, delimiter));
                            ch.pipeline().addLast(<span class="hljs-keyword">new</span> StringDecoder());
                            ch.pipeline().addLast(<span class="hljs-keyword">new</span> EchoClientHandler());

                        }

                    });

            <span class="hljs-comment">// 发起异步连接操作</span>
            ChannelFuture cf = boot.connect(host, port).sync();
            <span class="hljs-comment">// 等待客户端链路关闭</span>
            cf.channel().closeFuture().sync();

        } <span class="hljs-keyword">finally</span> {
            <span class="hljs-comment">// 优雅退出，释放NIO线程组</span>
            <span class="hljs-keyword">group</span>.shutdownGracefully();
        }

    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> throws Exception </span>{
        <span class="hljs-keyword">int</span> port = <span class="hljs-number">8080</span>;
        String host = <span class="hljs-string">"127.0.0.1"</span>;
        <span class="hljs-keyword">new</span> EchoClient().connect(port, host);
    }

    }
    `</pre>

    **客户端handler**

    <pre>`<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoClientHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ChannelHandlerAdapter</span> </span>{
    <span class="hljs-keyword">int</span> counter = <span class="hljs-number">0</span>;
    <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String ECHO_REQ = <span class="hljs-string">"Hi, krisjin. Welcome to Netty.$_"</span>;

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">EchoClientHandler</span><span class="hljs-params">()</span> </span>{

    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelActive</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> </span>{
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i < <span class="hljs-number">10</span>; i++) {
            ByteBuf write = Unpooled.copiedBuffer(ECHO_REQ.getBytes());

            ctx.writeAndFlush(write);

        }
    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        System.out.println(<span class="hljs-string">"This is "</span> + ++counter + <span class="hljs-string">"times recevie server : ["</span> + msg + <span class="hljs-string">"]"</span>);

    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelReadComplete</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        ctx.flush();
    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">exceptionCaught</span><span class="hljs-params">(ChannelHandlerContext ctx, Throwable cause)</span> </span>{
        cause.printStackTrace();
        ctx.close();
    }
    }
    