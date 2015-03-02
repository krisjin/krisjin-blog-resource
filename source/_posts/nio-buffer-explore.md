title: NIO Buffer理解
date: 2015-03-02 15:40:43
categories: NIO
tags: [buffer,NIO]
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Java Nio中的Buffer用于与Channel通道进行交互，数据从通道中读入缓冲区，或从缓冲器写入到通道中。缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO中的Buffer对象。  <!--more-->


先来看一段代码：



		public static void main(String[] args) {
		ByteBuffer buf = ByteBuffer.allocate(125);
		buf.put("Hello Buffer !".getBytes());

		System.out.println("Write mode:");
		System.out.println("Position: " + buf.position() + "; Limit: " + buf.limit() + "; Capacity: " + buf.capacity());

		buf.flip();

		System.out.println("Read mode:");

		System.out.println("Position: " + buf.position() + "; Limit: " + buf.limit() + "; Capacity: " + buf.capacity());

	}

输出结果：

		Write mode:
		Position: 14; Limit: 125; Capacity: 125
		Read mode:
		Position: 0; Limit: 14; Capacity: 125

上面的这段代码，是针对ByteBuffer的一些操作，allocate分配了指定大小的字节。然后向缓冲区中写入数据，查看Buffer中的状态，使用flip反转成读模式，查看Buffer中的状态。

下图理解：

![](/img/buffer explore.png)






