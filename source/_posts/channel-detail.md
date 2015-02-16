title: netty channel详解
date: 2015-02-16 08:00:35
categories: netty
tags: [netty]
---

### 1.Channel简述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;提起Channel大家并不会陌生，它是JDK的NIO类库的重要组成部分，就是提供了java.nio.SocketChannel 和java.nio.ServerSocketChannel,用于非阻塞的IO操作。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类似NIO的channel，Netty提供了自己的Channel及子类的实现，用于异步IO操作和其他相关的操作。
<!--more-->
### 2.Channel功能说明
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;io.netty.channel.channel是Netty网络操作抽象类，它聚合了一组功能，包括但不限于网络的读、写，客户端发起连接、主动关闭连接，链路关闭，获得通信双方的网络地址等。它也包含了Netty框架相关的一些功能，包括获取该Channel的EventLoop,获取缓冲分配器ByteBufAllocator和pipeline等。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面先从channel的接口分析，讲解它的主要API和功能，然后在了解下它的子类相关功能实现
### 3.Channel工作原理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Channel是Netty抽象出来的网络IO读写相关的接口，为什么不使用JDK NIO原生的Channel而要另起炉灶呢，主要原因如下：

1. JDK的SocketChannel和ServerSocketChannel没有提供统一的Channel接口供业务开发者使用，对于用户而言，没有统一的操作视图，使用起来并方便。
2. JDK的SocketChannel和ServerSocketChannel的主要职责就是网络操作，由于它们是SPI类接口，由具体的虚拟机厂家来提供，所以通过继承SPI功能类来扩展其功能的难度很大。直接实现ServerSocketChannle和SocketChannel抽象类，其工作量和重新开发一个新的Channeld的功能类差不多的。
3. Netty的channel需要能跟Netty整体框架融合在一起，例如IO模型、基于ChannelPipie的定制模型，以及基于元数据描述配置化的TCP参数等，这些JDK的SocketChannel和ServerSocketChannel都没有提供，需要重新封装。
4. 自定义的Channel，功能实现更加灵活。  

基于以上的4个原因，Netty重新设计了Channel接口，并且给予了很多不同的实现，它的设计原理很简单，但是功能却比较复杂，主要的设计理念如下：

1. 在Channel接口层，采用Facade模式进行统一封装，将网络IO操作，及相关联的的其它操作封装起来，统一对外提供。
2. Channel接口的定义尽量大而全，为SocketChannel和ServerSocketChannel提供统一的视图，由不同的子类实现不同的功能，公共功能在抽象父类实现，最大限度上实现功能和接口的重用。
3. 具体实现采用聚合而非包含的方式，将相关的功能类聚合在Channel中，由Channel统一负责分配和调度，功能实现更加灵活。

### 4.Channel功能介绍
Channel的功能比较繁杂，通过分类的方式对它主要的功能进行介绍。

##### 1、网络IO操作  
Channel网络IO相关的方法定义如下：

下面对这些API的功能进行分类说明，读写相关的API列表。  
1. **Channel read():** 从当前的Channel读取数据到第一个inbound缓冲区中，如果数据被成功读取，触发ChannelHandler.channelRead(ChannelHandlerContext,Object)事件，读取操作调用完之后，紧接着会触发ChannelHandler.channelReadComplete(ChannelHandlerContext)事件，这样业务的ChannelHandler可以决定是否需要继续读取数据。如果已经有读操作请求被挂起，则后续的读操作会被忽略。  
2. **ChannelFuture write(Object msg):**请求将当前的msg通过ChannePipe写入到目标Channel中。注意，write操作只是将消息存入到消息发送环形数组中，并没有真正被发送，只有调用flush操作才会被写入到Channel中，发送给对方。  
3. **ChannelFuture write(Object msg,ChannelPromise promise):**功能与write（Object msg）相同，但是携带了ChannelPromise 参数负责设置写入操作的结果。  
4. **ChannelFuture writeAndFlush(Object msg,ChannelPromise promiese):**与方法3功能类似，不同之处在于它会将消息写入Channel中发送，等价于单独调用write和flush操作的组合。  
5. **ChannelFuture writeAndFlush(Object msg):**功能等于方法4，但是没有ChannelPromise promiese 参数  
6. **Channel flush():** 将之前写入到发送环形数组中的消息全部写入到目标Channel中，发送给通信对方  
7. **ChannelFuture close(ChannelPromise promise):**主动关闭当前连接，通过ChannelPromise设置操作结果并进行结果通知，无论操作是否成功，都可以通过ChannelPromise获取操作结果。改操作会级联触发ChannelPipe中所有ChannelHandler的ChannelHandler.close(ChannelHandlerContext,ChannelPromise)事件。  
8. **ChannelFuture disconnect(ChannelPromise promise):**请求断开与远程通信对端的连接并使用ChannelPromise获取操作结果的通知信息。该方法会级联触发ChannelHandler.disconnect(ChannelHandlerContext,ChannelPromise)事件。  
9. **ChannelFuture connect(SocketAddress remoteAddress):**客户端使用指定的服务端地址发起连接请求，如果连接因为应答超时而失败，ChannelFuture中的操作结果ConnectTimeoutException异常，如果连接被拒绝，操作结果为ConnectException。该方法会级联触发ChannelHandler.connect(ChannelHandlerContext,SocketAddress,SocketAddress,ChannelPromise)事件。   
10. **ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress)：**功能与方法9类似，唯一不同的是先绑定本地地址localAddress，然后在连接服务端  
11. **ChannelFuture connect(SocketAddress remoteAddress, ChannelPromise promise):**与方式9功能类似，唯一不同的是多了一个ChannelPromise参数用于写入操作结果。  
12. **ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress, ChannelPromise promise)：**与方法11功能类似，唯一不同的就是绑定了本地地址。  
13. **ChannelFuture bind(SocketAddress localAddress)：**绑定指定的本地Socket地址，该方法会级联触发ChannelHandler.bind(ChannelHandlerContext,SocketAddress,ChannelPromise)事件。  
14. **ChannelFuture bind(SocketAddress localAddress, ChannelPromise promise)：**与13方法功能类似，多了一个ChannelPromise参数用于写入操作结果。  
15. ** ChannelConfig config()：**获取当前Channel的配置信息，例如CONNECT_TIMEOUT_MILLIS  
16. **blooean is opoen():**判断当前Channel是否已经打开  
17. **boolean isRegistered()：**判断当前Channel是否已经注册到EventLoop上。  
18. **boolean isActive()：**判断当前Channel是否已经处于激活状态  
19. **ChannelMetadata metadata()：**获取当前Channel的元数据描述信息，包括TCP参数配等。  
20. **SocketAddress localAddress()：**获取当前Channel的本地绑定地址  
21. **SocketAddress remoteAddress()**获取当前Channel通信的远程Socket地址。

##### 2、其它常用API功能说明  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一个比较重要的方法是eventLoop，Channel需要注册到EventLoop的多路复用器上，用于处理IO事件，通过eventLoop方法可以获取到Channel注册的EventLoop。EventLoop本质上就是处理网络读写事件的Reactor线程。在Netty中，它不仅仅用来处理网络事件，也可以用来执行定时任务和用户自定义NioTask等任务。   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第二个比较常用的方法是metadata方法，熟悉TCP协议的同学可能知道，当创建Socket的时候需要指定TCP参数，例如接收和发送的TCP缓冲区大小，TCP的超时时间，是否重用地址等等。在Netty中，每个Channel对应一个物理连接，每个连接都有自己的ＴＣＰ参数配置。所以，Ｃｈａｎｎｅｌ会聚合一个ChannelMetadata用来对TCP参数提供元数据描述信息，通过metadata方法就可以获取当前Channel的TCP参数配置。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第三个方法是parent,对于服务端Channel而言，它的父Channel为空，对于客户端Channel，它的父Channel就是创建它的ServerSocketChannel。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第四个方法是用户获取Channel标识的id,它返回ChannelId对象，ChannelId是Channel的唯一标识，**它的可能生成策略如下：**  
(1) 机器的MAC地址（EUI-48或者EUI-64）等可以代表全局唯一的信息。    
(2) 当前进程的ID。  
(3) 当前系统时间的毫秒——System.currentTimeMillis  
(4) 当前系统时间的纳秒——System.nanoTime
(5) 32位的随机整型数  
(6) 32位自增的序列数
 

### 5.Channel源码分析
Channel的实现类非常多，继承关系复杂，从学习的角度我们抽取最重要的两个NioServerSocketChannel和NioSocketChannel。  
服务端NioServerSocketChannel的继承关系类图如下：




#### AbstractChannel源码分析

1、**成员变量定义**  
在分析Abstract源码之前先了解下它的成员变量定义，首先定义了两个静态全局异常  

- CLOSED_CHANNEL_EXCEPTION 链路已经关闭已经异常  
- NOT_YET_CONNECTED_EXCEPTION 物理链路尚未建立异常  

声明完上述两个异常之后，通过静态块将它们的堆栈设置为空的EMPTY_STACK_TRACE。    estimatorHandle用于预测下一个报文的大小，它基于之前数据的采样进行分析预测。  
根据之前的Channel原理分析，我们知道AbstractChannel采用聚合的方式封装各种功能，从成员变量的定义可以看出，它聚合了一下内容。

- parent:代表了父类的Channel
- id：采用默认方式生成全局唯一的ID
- unsafe:Unsafe实例
- pipeline:当前Channel对应的DefaultChannelPipeline
- eventLoop:当前Channel注册的EventLoop

**代码清单**  
	 private static final InternalLogger logger = InternalLoggerFactory.getInstance(AbstractChannel.class);

    static final ClosedChannelException CLOSED_CHANNEL_EXCEPTION = new ClosedChannelException();
    static final NotYetConnectedException NOT_YET_CONNECTED_EXCEPTION = new NotYetConnectedException();

    static {
        CLOSED_CHANNEL_EXCEPTION.setStackTrace(EmptyArrays.EMPTY_STACK_TRACE);
        NOT_YET_CONNECTED_EXCEPTION.setStackTrace(EmptyArrays.EMPTY_STACK_TRACE);
    }

    private MessageSizeEstimator.Handle estimatorHandle;

    private final Channel parent;
    private final ChannelId id = DefaultChannelId.newInstance();
    private final Unsafe unsafe;
    private final DefaultChannelPipeline pipeline;
    private final ChannelFuture succeededFuture = new SucceededChannelFuture(this, null);
    private final VoidChannelPromise voidPromise = new VoidChannelPromise(this, true);
    private final VoidChannelPromise unsafeVoidPromise = new VoidChannelPromise(this, false);
    private final CloseFuture closeFuture = new CloseFuture(this);

    private volatile SocketAddress localAddress;
    private volatile SocketAddress remoteAddress;
    private final EventLoop eventLoop;
    private volatile boolean registered;

    /** Cache for the string representation of this channel */
    private boolean strValActive;
    private String strVal;



在此不一一列举，通过变量定义可以看出AbstractChannel聚合了所有Channel使用到的功能对象，由AbstractChannel提供初始化和统一封装，如果功能和子类强相关，则定义成抽象方法由子类具体实现。

2、**核心API源码分析**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先看下网络读写操作，前面有提到网络IO操作时讲到它会触发Channelpipeline中对应的事件方法，Netty基于事件驱动，我们也可以理解为当Channel进行IO操作时会产生对应的IO事件，然后驱动事件在ChannelPipe中传播，由相对应的ChannelHandler对事件进行拦截和处理，不关心的事件可以直接忽略。采用事件驱动的方式可以非常轻松的通过事件定义来划分事件拦截切面，方便业务的定制和功能扩展，相比AOP，其性能更高，但是功能却基本等价。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;网络IO操作直接调用DefaultPipelinec的相关方法，由DefaultChannelPipeline中对应的ChannelHandler进行具体的逻辑处理。   
**代码清单**（AbstractChannel网络IO操作实现）

	@Override
    public ChannelFuture bind(SocketAddress localAddress) {
        return pipeline.bind(localAddress);
    }

    @Override
    public ChannelFuture connect(SocketAddress remoteAddress) {
        return pipeline.connect(remoteAddress);
    }

    @Override
    public ChannelFuture connect(SocketAddress remoteAddress, SocketAddress localAddress) {
        return pipeline.connect(remoteAddress, localAddress);
    }

    @Override
    public ChannelFuture disconnect() {
        return pipeline.disconnect();
    }

    @Override
    public ChannelFuture close() {
        return pipeline.close();
    }

    @Override
    public Channel flush() {
        pipeline.flush();
        return this;
    }


AbstractChannel也提供了一些公共API的具体实现，例如localAddress和remoteAddress方法，源码如下

**代码清单**

	    @Override
    public SocketAddress localAddress() {
        SocketAddress localAddress = this.localAddress;
        if (localAddress == null) {
            try {
                this.localAddress = localAddress = unsafe().localAddress();
            } catch (Throwable t) {
                // Sometimes fails on a closed socket in Windows.
                return null;
            }
        }
        return localAddress;
    }

    protected void invalidateLocalAddress() {
        localAddress = null;
    }

    @Override
    public SocketAddress remoteAddress() {
        SocketAddress remoteAddress = this.remoteAddress;
        if (remoteAddress == null) {
            try {
                this.remoteAddress = remoteAddress = unsafe().remoteAddress();
            } catch (Throwable t) {
                // Sometimes fails on a closed socket in Windows.
                return null;
            }
        }
        return remoteAddress;
    }


首先从缓存的成员变量中获取，如果第一次调用为空，需要通过unsafe的remoteAddress获取，它是个抽象方法，具体由对应的channel子类实现。

#### AbstractNioChannel源码分析

1、**成员变量定义**  
首先还是先从成员变量定义入手，来了解下它的功能实现，成员变量如下：  
**代码清单**

	private static final InternalLogger logger =
            InternalLoggerFactory.getInstance(AbstractNioChannel.class);

    private final SelectableChannel ch;
    protected final int readInterestOp;
    private volatile SelectionKey selectionKey;
    private volatile boolean inputShutdown;

    /**
     * The future of the current connection attempt.  If not null, subsequent
     * connection attempts will fail.
     */
    private ChannelPromise connectPromise;
    private ScheduledFuture<?> connectTimeoutFuture;
    private SocketAddress requestedRemoteAddress;


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于NIO Channel 、NioSocketChannel 和 NioServerSocketChannel需要共用，所以定义了一个java.nio.SocketChannel和java.nio.ServerSocketChannel的公共父类SelectableChannel,用于设置SelectableChannel参数和进行IO操作。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;readInterestOp代表了Jdk SelectionKey的OP_READ。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;volatile修饰的selectionKey，该SelectionKey是Channel注册到EventLoop后返回的选择键。由于Channel会面临多个业务线程的并发写操作，当SelectionKey由SelectionKey修改之后，为了能让其它线程感知到变比，所以需要使用volatile保证修改的可见性。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最后定义了代表连接操作结果的ChannelPromise以及连接超时定时器ScheduledFuture和请求通信的地址信息。 



1、**核心API源码解析** 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先看下在AbstractNioChannel实现的主要API首先是Channel的注册。  
**代码清单**

	 @Override
    protected void doRegister() throws Exception {
        boolean selected = false;
        for (;;) {
            try {
                selectionKey = javaChannel().register(eventLoop().selector, 0, this);
                return;
            } catch (CancelledKeyException e) {
                if (!selected) {
                    // Force the Selector to select now as the "canceled" SelectionKey may still be
                    // cached and not removed because no Select.select(..) operation was called yet.
                    eventLoop().selectNow();
                    selected = true;
                } else {
                    // We forced a select operation on the selector before but the SelectionKey is still cached
                    // for whatever reason. JDK bug ?
                    throw e;
                }
            }
        }
    }

定义一个布尔类型的局部变量selected 来表示注册操作是否成功，调用SelectableChannel的register方法，将当前的Channel注册到EventLoop的多路复用器上，SelectableChannel的注册方法定义如下：

**代码清单**（JDK SelectableChannel的注册方法定义）

	 public abstract SelectionKey register(Selector sel, int ops, Object att)
        throws ClosedChannelException;

注册Channel的时候需要指定监听的网络操作位来表示Channel对哪几类网络事件感兴趣，具体定义如下：

- public static final int OP_READ = 1 << 0  读操作位
- public static final int OP_WRITE = 1 << 2 写操作位
- public static final int OP_CONNECT = 1 << 3 客户端连接服务端操作位
- public static final int OP_ACCEPT = 1 << 4 服务端接收客户端连接操作位

AbstractNioChannel注册的是0，说明对任何事件都不敢兴趣，仅仅完成注册操作。注册的时候可以指定附件，后续Channel接收到网络事件通知时可以从SelectionKey中重新获取之前的附件进行处理，此处将AbstractNioChannel的实现子类自身当作附件注册。如果注册Channel成功，则返回selectionkey,通过selectionkey可以从多路复用器中获取Channel对象。
