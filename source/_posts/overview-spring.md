title: 初识Spring
date: 2015-02-16 16:29:32
categories: spring
tags:
---

### 概述
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring开源框架是一个轻量级的解决方案和一站式的构建企业级应用程序，Spring框架是模块化的，允许你使用需要的功能模块。你可以用IOC容器、strust、hibernate等，也可以选择不用，它支持声明式事务管理，使用RMI远程调用或Web Services,提供了优雅的MVC框架,也可以集成AOP到你的应用中。
<!--more-->

### 介绍
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring框架提供了完善的基础架构来支撑快速开发Java应用，有了这样完善的基础架构，开发人员就可以专注于应用的业务开发。Spring是非侵入性，针对Java POJO的开发。

#### 整体架构

下图是Spring架构图：  
![](/img/spring-overview.png)
1、**Core Container**  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;核心容器由**Beans、Core、Context**和**ExpressionLanguage**模块组成。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Beans** 和 **Core**模块是框架的基础部分，包括了IoC(控制反转)和DI（依赖注入）特性。BeanFactory是一个复杂的工厂模式实现，它去除了编程中实现单例，允许通过配置指定依实际编程业务逻辑。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Context** 模块构建在**Beans**和**Core**模块提供的坚实基础之上。它是访问对象的框架式的方式的装置，它类似于一个JNDI注册，**Context**模块继承了**Beans**模块的特性，并增加了对国际化的支持。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Expression Language**提供了强大的表达式语言，在运行时查询和操作对象图


2、**Data Access/Integration**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据访问层集成层由JDBC、ORM、OXM、JMS和事务模块

3、**Web**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Web层由Web、Web-Servlet、Web-Struts和Web-portlet模块组成。

4、**AOP and Instrumentation**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring's AOP module provides an AOP Alliance-compliant aspect-oriented programming implementation allowing you to define, for example, method-interceptors and pointcuts to cleanly decouple code that implements functionality that should be separated. Using source-level metadata functionality, you can also incorporate behavioral information into your code, in a manner similar to that of .NET attributes.
