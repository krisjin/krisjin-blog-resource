title: Java GC日志图解
date: 2015-09-06 14:51:36
categories: Jvm
tags: [GC,Jvm]
---

配置JVM 参数：

1. **-XX:+PrintGCDetails**   打印GC回收的细节
2. **-XX:+PrintGCTimeStamps** 打印GC停顿耗时
3. **-Xloggc:gc.log**  GC日志输出到文件



**YoungGC日志：**
![](/img/yonggc.jpg)

**FullGC日志：**

![](/img/fullgc.jpg)

转载自：[http://www.chinasb.org/archives/2012/09/4921.shtml](http://www.chinasb.org/archives/2012/09/4921.shtml "http://www.chinasb.org/archives/2012/09/4921.shtml")