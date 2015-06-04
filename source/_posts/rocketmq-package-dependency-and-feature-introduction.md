title: RocketMQ包依赖关系与功能介绍
date: 2015-06-04 22:59:22
categories: rocketmq
tags: [rocketmq,mq]

---

rocketmq包含9个子模块：

**rocketmq-common**：通用的常量枚举、基类方法或者数据结构，按描述的目标来分包通俗易懂。包名有：admin，consumer，filter，hook，message等。

**rocketmq-remoting**：用Netty4写的客户端和服务端，fastjson做的序列化，自定义二进制协议。

**rocketmq-srvutil**：只有一个ServerUtil类，类注解是，只提供Server程序依赖，目的为了拆解客户端依赖，尽可能减少客户端的依赖。

**rocketmq-store**：存储服务，消息存储，索引存储，commitLog存储。

**rocketmq-client**:客户端，包含producer端和consumer端，发送消息和接收消息的过程。

**rocketmq-filtersrv:**消息过滤器server，现在rocketmq的wiki上有示例代码及说明，https://github.com/alibaba/RocketMQ/wiki/filter_server_guide

**rocketmq-broker**：对consumer和producer来说是服务端，接收producer发来的消息并存储，同时consumer来这里拉取消息。

**rocketmq-tools**：命令行工具。

**rocketmq-namesrv**：NameServer，类似SOA服务的注册中心，这里保存着消息的TopicName，队列等运行时的meta信息。一般系统分dataNode和nameNode，这里是nameNode。

如下依赖图：
![](/img/rocketmq-pkg-den.png)



参考： [考拉哥的博客](http://lifestack.cn/archives/324.html "http://lifestack.cn/archives/324.html") 