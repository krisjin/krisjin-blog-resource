title: Spring注入之道
date: 2015-02-21 09:34:25
categories: spring
tags:
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;各个业务对象之间相互协作来完成同一使命。这时，各个业务对象的相互依赖就是无法避免的。对象之间需要相互协作，在横向上它们存在一定的依赖性。而现在我们就是要看一下，在Spring的IoC容器的XML配置中，应该如何表达这种依赖型。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;既然业务对象现在都符合IoC的规则，那么要了解的表达方式其实很简单，无非就是看下构造方法注入和setter方法注入通过XML是如何表达的而已。<!--more-->  

### 构造方法注入
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;按照Spring的IoC容器配置格式，通过构造方法注入方式，为当前业务对象注入其所依赖的对象，需要使用*constructor-arg*,如下代码所示：

	<bean id="newsPanel" class="org.spring.explore.controller.NewsPanel">
		<constructor-arg>
			<ref bean="newsService" />
			<ref bean="statNewsService" />
		</constructor-arg>
	</bean>
	<bean id="newsService" class="org.spring.explore.service.NewsService" />
	<bean id="statNewsService" class="org.spring.explore.service.StatNewsService" />

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有些时候，容器在加载XML配置的时候，因为某些原因，无法明确配置项与对象的构造方法参数列表的的对应关系，就需要请*constructor-arg*的type和index属性出马。比如对象存在多个构造方法，当参数列表数目相同而类型不同的时候，容器无法区分应该使用哪个构造方法来实例化对象，或者构造方法可能同时传入最少两个类型的对象。

#### type属性

	<bean id="statNewsService" class="org.spring.explore.service.CacheService">
		<constructor-arg type="String">
			<value>memcached</value>
		</constructor-arg>
		<constructor-arg type="int">
			<value>123</value>
		</constructor-arg>
	</bean>

#### index属性

	<bean id="statNewsService" class="org.spring.explore.service.CacheService">
		<constructor-arg index="1">
			<value>memcached</value>
		</constructor-arg>
		<constructor-arg index="0">
			<value>123</value>
		</constructor-arg>
	</bean>


在构造函数注入中使用ref来引用容器中其它的对象实例，可以通过ref的bean、parent属性来指定引用的对象是什么,如下所示

	<bean id="fxNews" class="...">
		<constructor-arg>
			<ref bean="" />
		</constructor-arg>
		<constructor-arg>
			<ref parent="" />
		</constructor-arg>
	</bean>
parent只能指定位于当前容器的父容器中定义的对象引用。  
bean 则通吃，直接使用bean来指定对象引用就行。  

BeanFactory可以分层次（通过实现HierarchicalBeanFactory接口），容器A在初始化的时候，可以首先加载容器B中所有的对象定义，然后在加载自身的对象定义，这样，容器B就成了容器A的父容器，容器A可以引用容器B中的所有对象定义。

### setter方法注入
---
	<bean id="fxNews" class="...">
		<property name="newsService" ref="newsService"/>
		<property name="statNewsService" ref="statNewsService"/>
	</bean>

