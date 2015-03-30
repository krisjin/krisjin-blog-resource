title: Guice module研究
date: 2015-03-31 00:08:14
categories: Guice
tags: [guice]
---

模块化的配置信息，显然是基于接口的绑定。使用Injector实例
<!--more-->



	  A module contributes configuration information, typically interface
	  bindings, which will be used to create an {@link Injector}. A Guice-based
	  application is ultimately composed of little more than a set of
	  {@code Module}s and some bootstrapping code.
	 
	  <p>Your Module classes can use a more streamlined syntax by extending
	  {@link AbstractModule} rather than implementing this interface directly.
	 
	  <p>In addition to the bindings configured via {@link #configure}, bindings
	  will be created for all methods annotated with {@literal @}{@link Provides}.
	  Use scope and binding annotations on these methods to configure the
	  bindings.


类结构：

![](/img/am.png)


![](/img/abstractmodule.jpg)