title: NIO多路复用器理解
date: 2015-03-19 22:31:19
categories: NIO
tags:
---

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。<!--more-->

服务端通信序列图
![](/img/nio-selector.png)