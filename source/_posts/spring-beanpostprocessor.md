title: Spring bean实例化后处理器
date: 2015-02-23 07:38:24
categories: spring
tags:
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Spring Bean 实例化阶段后处理器主要是BeanPostProcessor接口，它与BeanFactoryPostProcessor容易产生混淆。只要记住BeanPostProcessor是存在于对象实例化阶段，而BeanFactoryPostProcessor则存在于容器启动阶段。这两个概念就比较容易区分了。<!--more-->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;与BeanFacotryPostProcessor通常会处理容器所有符合条件的BeanDefinition类似，BeanPostProcessor会处理容器内所有符合条件的实例化后的对象实例。该接口声明了两个方法，分别在两个不同的时机执行，见下代码定义：

	public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one; if
	 * {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one; if
	 * {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

	}

*postProcessBeforeInitialization*方法就如下图中BeanPostProcess前置处理这一步将会执行的方法，*postProcessAfterInitialization*方法对应下图中BeanPostProcessor后置处理那一步执行的方法。BeanPostProcessor的两个方法中都传入了原理对象实例的引用，这为我们扩展容器的对象实例化过程中的行为提供了极为的便利。我们几乎可以对传入的对象实例执行任何的操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通常比较常见的使用BeanPostProcessor的场景，是处理标记接口实现类，或者为当前对象提供代理实现。ApplicationContext对应的那些Aware接口实际上是通过BeanPostProcessor的方式进行处理的。当ApplicationContext中每个对象实例化过程走到BeanPostProcessor前置处理这一步是，ApplicationContext容器会检测到之前注册到容器的ApplicationContextAwareProcessor这个BeanPostProcessor实现类，然后调用其*postProcessBeforeInitialization*方法，检查并设置Aware相关依赖。



**Bean的实例化过程**

![](/img/Bean的实例化过程.png)



**BeanPostProcessor类图**  

![](/img/beanpostprocessor.png)