title: Jetty使用介绍
date: 2015-06-16 22:14:30
categories: jetty
tags: [jetty]
---


Jetty的使用方式有很多种，可以在应用程序中嵌入，可以从不同的构建系统中去启动，构建在不同的JVM之上，或是作为一个独立的服务。适合部署Web应用程序独立的分布。

## Jetty下载

[下载地址](http://download.eclipse.org/jetty/ "http://download.eclipse.org/jetty/")



## Jetty目录介绍


|Location	|Description|
|-----------|-----------|
|license-eplv10-aslv20.html	|License文件|
|README.txt	|开始入门信息|
|VERSION.txt	|版本信息|
|bin/	|linux环境运行shell|
|demo-base/	|demo|
|etc/	|jetty xml配置文件|
|lib/	|jetty运行需要的所有文件|
|logs/	|日志目录|
|modules/	|模块定义目录|
|notice.html	|许可证信息和异常|
|resources/	|Directory containing additional resources for classpath, activated via configuration|
|start.d/	|Directory of *.ini files containing arguments that are added to the effective command line (see start.ini)|
|start.ini	|File containing the arguments that are added to the effective command line (modules, properties and XML configuration files)|
|start.jar	|调用jetty执行jar|
|webapps/	|Directory containing webapps that run under the default configuration of Jetty|