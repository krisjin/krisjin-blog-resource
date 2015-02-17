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
1、**Spring核心容器**  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;核心容器由**Beans、Core、Context**和**ExpressionLanguage**模块组成。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Beans** 和 **Core**模块是框架的基础部分，包括了IoC(控制反转)和DI（依赖注入）特性。BeanFactory是一个复杂的工厂模式实现，它去除了编程中实现单例，允许通过配置指定依实际编程业务逻辑。 BeanFactory使用控制反转(IOC)模式，将应用程序的配置和依赖性规范与实际的应用程序代码分开 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Context** 模块构建在 **Beans** 和**Core**模块提供的坚实基础之上。**Context**模块继承了**Beans**模块的特性，Spring上下文(Context)其实是一个配置文件，用于向Spring框架提供上下文信息。Spring上下文包括企业服务，如JNDI，EJB,电子邮件，国际化，校验和调度功能。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Expression Language**提供了强大的表达式语言，在运行时查询和操作对象图


2、**Spring DAO**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ＤＡＯ抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且很大程度地降低了需要编写的大量异常代码（如打开和关闭数据库连接代码）。SpringDAO的面向JDBC的异常也遵从通用的DAO异常层次结构

3、**Spring Web**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Web 上下文模块建立在应用程序上下文模块之上，为基于Web的应用程序提供了上下文。所以，Spring 框架支持与以Struts为首的各种MVC框架的集成。Web模块还简化了处理多部分请求，以及将请求参数绑定到域对象的工作。

4、**Spring AOP**  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过配置管理特性，SpringAOP模块直接将面向方面的编程功能，集成到了Spring框架中。所以，可以很方便地使Spring框架管理的任务对象支持AOP。SpringAOP模块为基于Spring的应用程序中的对象提供了事务管理服务。通过使用SpringAOP，不用依赖EJB组件，就可以将声明性事务管理集成到应用程序中。它也是Spring所具有的独特组件，同时也是最核心的组成部分之一.
