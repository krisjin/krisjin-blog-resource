title: jcmd工具使用
date: 2015-09-02 11:31:49
categories: Java
tags: [调优]
---

## jcmd概述
在jdk1.7推出后，新增了一个jcmd诊断命令行工具，它是一个多功能的工具，可以导出堆栈，查看jvm进程，导出线程信息，执行GC等。

## 示例

- 列出当前运行的所有JVM

		jcmd -l

![](/img/jcmd01.jpg)  

- 列出指定虚拟机支持的所有命令

		jcmd 5052 help

![](/img/jcmd02.jpg)

- 查看虚拟机启动时间

		jcmd 35199 VM.uptime

![](/img/jcmd03.jpg)

- 打印线程栈信息

	jcmd 5052 Thread.print

![](/img/jcmd04.jpg)

- 查看系统中类统计信息

		jcmd -5052 GC.class_histogram

![](/img/jcmd05.jpg)

- 导出堆信息

 	jcmd 5052 GC.heap_dump /opt/dump.txt  

![](/img/jcmd06.jpg)

- 获取VM启动参数

		jcmd 35199 VM.flags

![](/img/jcmd07.jpg)

- 获取所有性能相关数据

		jcmd 35199 PerfCounter.print

![](/img/jcmd08.jpg)  

![](/img/jcmd09.jpg)


## 参考
[http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/JavaJCMD/index.html](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/JavaJCMD/index.html "http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/JavaJCMD/index.html")