title: 对比 Redis 与 Memcached
date: 2015-10-12 10:41:11
categories: NoSQL
tags: [redis,memcached]
---

前几天， Redis 的作者 Antirez 写了[一篇博客](http://antirez.com/news/94)， 驳斥了某个库作者认为 Redis 比不上 Memcached 的观点。

Antirez 的博文列举了几个他认为 Redis 比 Memcached 更优秀的地方， 但是并没有对 Redis 和 Memcached 的每个功能进行详细的对比， 而这篇文章要做的就是对 Antirez 的文章进行补充： 本文将从功能和网络搜索热度两个方面， 对 Redis 和 Memcached 进行详细的对比， 通过查看这些对比结果， 读者应该能明白 Redis 和 Memcached 之间的区别。

# 功能对比

|对比项目|Memcached|Redis|
|-------|---------|-----|
|支持的数据结构|字符串（二进制安全，可直接储存字节数据）|字符串（二进制安全，可直接储存字节数据）    (散列,列表,集合,有序集合,位图（bitmap）,地理位置（GEO）,HyperLogLog)|
|单机附加功能|自动过期,流水线|自动过期,流水线,事务,Lua 脚本,发布与订阅,键空间通知,AOF 持久化,RDB 持久化|
|多机附加功能|由客户端实现的，基于分片的集群（无Sentinel或复制），通过 twemproxy 实现分片|由服务器端实现的，基于分片的集群，自带Sentinel和复制,复制（无需集群，可独立运作）,Sentinel（无需集群，可独立运作）,twemproxy、codis、redis-cerberus 等多种第三方代理可选|
|内存分配方式|slab|jemalloc|
|网络模型|使用多个线程处理多个客户端，使用锁对线程进行同步|单线程，通过 I/O 多路复用来处理多个客户端|


# 网络搜索热度

Google 趋势，全球： https://www.google.com/trends/explore#q=Redis%2C%20Memcached&cmpt=q&tz=Etc%2FGMT-8  
![](/img/google-trend-global.png)

Google 趋势，中国： https://www.google.com/trends/explore#q=Redis%2C%20Memcached&geo=CN&cmpt=q&tz=Etc%2FGMT-8  
![](/img/google-trend-china.png)

百度指数，中国： http://index.baidu.com/?tpl=trend&word=redis%2Cmemcached  
![](/img/baidu-zhishu.png)

# 结语

在看过了上文的对比之后， 读者应该能够明白 Redis 和 Memcached 之间有什么区别了。

因为笔者并不是特别熟悉 Memcached ， 所以如果你发现本文中关于 Memcached 的描述有任何不正确的地方， 又或者我遗漏了 Memcached 的某项功能， 那么欢迎各位对文章进行补充。

黄健宏
2015.10.1

原文链接：[http://blog.huangz.me/diary/2015/comparison-of-redis-and-memcached.html?utm_source=tuicool](http://blog.huangz.me/diary/2015/comparison-of-redis-and-memcached.html?utm_source=tuicool)