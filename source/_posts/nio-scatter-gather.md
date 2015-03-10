title: NIO Scatter/Gather实例
date: 2015-03-10 12:00:56
categories: NIO
tags: [nio,scatter,gather]
---


Java NIO开始支持scatter/gather，scatter/gather用于描述从Channel（译者注：Channel在中文经常翻译为通道）中读取或者写入到Channel的操作。
分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。
聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。<!--more-->

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

Scatter/Gather操作查看下图：

![](/img/scatter-gather.png)



**代码清单：**

	import java.io.FileNotFoundException;
	import java.io.IOException;
	import java.io.RandomAccessFile;
	import java.nio.ByteBuffer;
	import java.nio.channels.FileChannel;
	import java.nio.channels.GatheringByteChannel;
	import java.nio.channels.ScatteringByteChannel;

	public class ScatterAndGather {

	static String headers = "header:13byte ";
	static String bodys = "body:Scattering and Gathering example";

	public static void main(String[] args) {
		gather();
		scatter();
	}

	public static void scatter() {

		ByteBuffer header = ByteBuffer.allocate(13);
		ByteBuffer body = ByteBuffer.allocate(100);

		ByteBuffer[] bufferArray = { header, body };

		ScatteringByteChannel channel = getChannel();

		try {
			channel.read(bufferArray);
		} catch (IOException e) {
			e.printStackTrace();
		}

		header.rewind();
		body.rewind();

		String headerStr = convertBufferToString(header);
		String bodyStr = convertBufferToString(body);
		System.out.println(headerStr);
		System.out.println(bodyStr);

	}

	public static void gather() {
		ByteBuffer header = ByteBuffer.allocate(14);
		ByteBuffer body = ByteBuffer.allocate(100);

		header.put(headers.getBytes());
		body.put(bodys.getBytes());

		GatheringByteChannel channel = getChannel();

		try {
			header.flip();
			body.flip();
			channel.write(new ByteBuffer[] { header, body });
		} catch (IOException e) {
			e.printStackTrace();
		}

	}

	public static FileChannel getChannel() {

		RandomAccessFile raf = null;
		try {
			raf = new RandomAccessFile("e:/sg.txt", "rw");
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		}

		return raf.getChannel();
	}

	public static String convertBufferToString(ByteBuffer buffer) {
		StringBuilder sb = new StringBuilder();
		while (buffer.remaining() != 0) {
			sb.append((char) buffer.get());
		}
		return sb.toString();
	}
	
	}