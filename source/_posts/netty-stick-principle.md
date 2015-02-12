title: LineBasedFrameDecoder和StringDecoder的原理分析
date: 2015-02-10 16:56:24
categories: netty
tags: [netty]
---
### LineBasedFrameDecoder详解

LineBasedFrameDecoder的工作原理是依次遍历ByteBuf中的可读字节，判断是否有“\n”或者“\r\n”，如果有就以此为结束为止，从可读索引到结束位置区间的字节组成了一行。它是以换行符为结束标志的解码器，支持携带结束符或者不携带结束符两种解码方式，同时支持单行的最大长度。如果连续读取到最大长度后仍然没有发现换行符，就回抛出异常，同时忽略掉之前读到的异常码流。
<!--more-->
*关键实现代码：*

	  private static int findEndOfLine(final ByteBuf buffer) {
        final int n = buffer.writerIndex();
        for (int i = buffer.readerIndex(); i < n; i ++) {
            final byte b = buffer.getByte(i);
            if (b == '\n') {
                return i;
            } else if (b == '\r' && i < n - 1 && buffer.getByte(i + 1) == '\n') {
                return i;  // \r\n
            }
        }
        return -1;  // Not found.
    }


### StringDecoderr详解

StringDecoder的功能非常简单，就是将接收到的对象转换成字符串，然后继续调用后面的handler。

*关键代码：*

	   public String toString(int index, int length, Charset charset) {
        if (length == 0) {
            return "";
        }

        ByteBuffer nioBuffer;
        if (nioBufferCount() == 1) {
            nioBuffer = nioBuffer(index, length);
        } else {
            nioBuffer = ByteBuffer.allocate(length);
            getBytes(index, nioBuffer);
            nioBuffer.flip();
        }

        return ByteBufUtil.decodeString(nioBuffer, charset);
    }
