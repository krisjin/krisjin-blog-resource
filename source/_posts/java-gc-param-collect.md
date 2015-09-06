title: Java GC 参数使用
date: 2015-09-06 15:48:34
categories: Jvm
tags: [GC,Jvm]
---

### Java GC 参数使用

- **-verbose:gc** 简单显示GC收集的信息

- **-XX:+PrintGC** 简单GC日志模式，为每一次新生代（young generation）的GC和每一次的Full GC打印一行信息
 
- **-XX:+PrintGCDetails** 打印GC回收的细节
 
- **-XX:+PrintGCTimeStamps**  打印GC停顿耗时 （表示自JVM启动至今的时间戳会被添加到每一行中）

- **-xx:+PrintHeapAtGC**  打印队的更详细信息
 
- **-Xloggc:gc.log GC** 日志输出到文件

- **-XX:+PrintGCApplicationStoppedTime**   输出GC造成应用暂停的时间

- **-XX:+PrintTenuringDistribution**  打印对象的存活期限信息。

> Total time for which application threads were stopped: 3.0888536 seconds