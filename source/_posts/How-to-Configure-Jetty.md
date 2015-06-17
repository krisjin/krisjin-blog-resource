title: 怎么配置Jetty
date: 2015-06-17 13:47:20
categories: jetty
tags: [jetty]
---


## Jetty POJO 配置

Jetty的核心组件简单的就像一个POJO一样，Jetty的配置过程大部分是实例化的过程，在POJO之上的装配和设置域操作，你可以通过下面的方式实现：

1. 直接写代码实例化和装配Jetty对象，如：[Embedding Jetty](http://www.eclipse.org/jetty/documentation/9.2.8.v20150217/embedding-jetty.html "http://www.eclipse.org/jetty/documentation/9.2.8.v20150217/embedding-jetty.html")
2. 使用Jetty xml配置，这是一个IOC框架，实例化和装配Jetty对象作为XML对象，etc/jetty.xm 是主要的Jetty配置文件。但是也有其它配置jetty特定配置文件在发布版本里。
3. 用第三方的IOC框架，如Spring，实例化和装配Jetty对象作为一个Spring Bean。

因为主要的Jetty配置文件都是IOC来去完成的，可以去参考[Jetty API文档](http://download.eclipse.org/jetty/stable-9/apidocs/ "http://download.eclipse.org/jetty/stable-9/apidocs/")。


## Jetty Start配置文件

Jetty使用下面的配置文件去实例化、注入和启动服务器通过执行start.jar

**ini文件**：

Jetty使用命令行的启动方式，**start.ini** 和start.d/*.ini下的文件可以创建一有效的命令行参数，参数可能是：

- Xml文件使用jetty IOC的文件格式
- 模块激活使用 --module=name配置
- 属性配置方式 name=value,参数化使用jetty IOC xml
- 一个标准的Java属性文件包含额外的 start文件属性
- 其它start.jar选项（参见：java -jar start.jar --help ）
- JVM 选项

**mod文件**：  
modules/*.mod文件包含模块定义，激活方式 --module=name 名称是每个模块文件的定义

- 模块依赖按照顺序激活
- 所需的库模块添加到类路径中
- 该模块所需要的XML文件添加到有效的命令行
- 文件所需的激活模块
- 模板ini文件 --add-to-start =name 激活时使用的名字


**XML文件**：  
Jetty或spring IoC格式的XML文件或在命令行上列出,ini文件或添加到模块定义有效的命令行。XML文件实例化和注入实际的Java对象,包括服务器、连接器和上下文。因为码头奥委会XML文件经常使用属性,许多常见配置任务可以没有编辑这些XML文件来完成。如果需要XML配置更改,应该从jetty复制XML文件 jetty.home/etc 到 jetty.base/etc 在修改之前


## 其它配置文件







## Jetty IoC XML 配置格式