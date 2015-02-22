title: Spring Xml配置文件之beans
date: 2015-02-20 18:43:48
categories: spring
tags:
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上文中我们介绍了几种注入的方式，xml格式的容器信息管理方式是Spring提供的最为强大、支持最为全面的方式。所用使用XML文件进行配置信息加载的Spring IoC容器，包括BeanFactory和ApplicationContext的所有XML相应实现，都使用统一的XML格式。  <!--more-->  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所有注册到容器的业务对象，在Sring中称之为Bean,所以，每个对象在XMl中的映射也自然而然的对应一个*bean*的元素。既然容器中可以管理很多的业务对象，那么把这么多的bean元素组织起来，就叫做beans。下面看下beans元素。

  
### beans元素属性
1. **default-lazy-init**：其值可以指定为true和false,默认值是false。用来标识是否对所有的bean进行延迟初始化。
2. **default-autowire**：可以取值为no、byName、byType、constructor、default，默认为no。
3. **default-init-method**：如果所管理的bean按照某种规则，都有同样名称的初始化方法的话，可以在这里统一指定这个初始化方法名，而不用在没个bean上都重复单独指定。
4. **default-destroy-method**： 与default-init-method相对应，如果所管理的bean有按照某种规则使用了相同名称的对象销毁方法，可以通过这个属性统一指定。

### import元素
通常情况下，可以根据模块功能或者层次关系，将配置信息分门别类地放到多个配置文件中。在加载主配置文件时，并将主配置文件所依赖的配置文件同时加载，可以在这个主配置文件中通过import元素对其所依赖的配置文件进行引用。


