title: protobuf入门
date: 2015-02-23 11:12:36
categories: protobuf
tags:
---

### protobuf简介
protobuf是google提供的一个开源序列化框架，类似于XML，JSON这样的数据表示语言，其最大的特点是基于二进制，因此比传统的XML表示高效短小得多。虽然是二进制数据格式，但并没有因此变得复杂，开发人员通过按照一定的语法定义结构化的消息格式，然后送给命令行工具，工具将自动生成相关的类，可以支持java、c++、python等语言环境。通过将这些类包含在项目中，可以很轻松的调用相关方法来完成业务消息的序列化与反序列化工作。<!--more-->

protobuf在google中是一个比较核心的基础库，作为分布式运算涉及到大量的不同业务消息的传递，如何高效简洁的表示、操作这些业务消息在google这样的大规模应用中是至关重要的。而protobuf这样的库正好是在效率、数据大小、易用性之间取得了很好的平衡。

### protobuf 限定修饰符介绍：

1. required必须的字段，如果不赋值就会抛出 com.google.protobuf.UninitializedMessageException: Message missing required fields: ...异常
2. optional可选字段，没什么好说的就是可有可无咯
3. repeated可重复的字段可以用来表示数组，在这里我还小小的纠结了会，搞过去就好了（纠结了好一会才知道protobuf数组怎么定义）。其实定义数组很简单repeated string name=字段号;然后在赋值的刚开始用数组的形式来赋值，会抛出java.lang.IndexOutOfBoundsException: Index: 0, Size: 0的异常，我在想size为0，也就是说不能这样搞呀，然后看了下源码是com.google.protobuf.LazyStringList name_ = com.google.protobuf.LazyStringArrayList.EMPTY这样的，也就是说这个玩儿就是个集合嘛，所以赋值就用集合那套来搞定好了

