title: Spring容器之BeanFactory
date: 2015-02-20 18:08:30
categories: spring
tags:
---

Spring核心容器中实现了控制反转与依赖注入，它解决了业务类依赖关系的隔离，所有的对象创建及依赖关系的绑定都有Spring容器实现，由它统一管理。  <!--more-->
### BeanFactory的对象注册和依赖绑定
BeanFactory 作为一个IoC Service Provider,为了能够明确管理各个业务对象以及业务对象之间的依赖绑定关系，同样需要某种途径来记录和管理这写信息。
1). **直接编码方式**

代码清单：

	public class HardCodeBeanFactory {

	public static void main(String[] args) {
		DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
		BeanFactory container = bind(factory);
		UserController uc = (UserController) container.getBean("userController");
		uc.addUser(new User());

	}

	public static BeanFactory bind(BeanDefinitionRegistry registry) {
		AbstractBeanDefinition controller = new RootBeanDefinition(UserController.class, true);
		AbstractBeanDefinition userService = new RootBeanDefinition(UserService.class, true);
		
		//将Bean注册到容器之中
		registry.registerBeanDefinition("userController", controller);
		registry.registerBeanDefinition("userService", userService);
		
		//指定依赖关系（setter方法注入方式）
		MutablePropertyValues propertyVlaue = new MutablePropertyValues();
		propertyVlaue.addPropertyValue(new PropertyValue("userService", userService));
		controller.setPropertyValues(propertyVlaue);
		
		//构造函数注入方式
		// ConstructorArgumentValues argValue= new ConstructorArgumentValues();
		// argValue.addIndexedArgumentValue(0, userService);
		// controller.setConstructorArgumentValues(argValue);

		return (BeanFactory) registry;

	}
	}


2). Xml配置方式

代码清单：

	public class ConfigFileBeanFactory {
	public static void main(String[] args) {

		// ApplicationContext factory = new
		// ClassPathXmlApplicationContext("classpath:applicationContext.xml");

		DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

		BeanFactory container = (BeanFactory) bind(beanFactory);

		UserController userController = (UserController) container.getBean("userController");

		userController.addUser(new User());

	}

	public static BeanFactory bind(BeanDefinitionRegistry registry) {

		XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(registry);
		reader.loadBeanDefinitions("classpath:applicationContext.xml");

		return (BeanFactory) registry;
	}
	}


3).注解方式

代码清单：

	public class AnnotationInjection {

	public static void main(String[] args) {
		BeanFactory factory = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
		AnnotationUserController userController = (AnnotationUserController) factory.getBean("userController");
		userController.addUser(new User());
	}

	}

---

	@Component("userController")
	public class AnnotationUserController {

	@Autowired
	private UserService userService;

	public void addUser(User user) {
		userService.addUser(new User());
	}

}


配置文件：

	<context:component-scan base-package="org.spring.explore"></context:component-scan>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在AnnotationUserController类中增加了两个注解，@Autowired是这里的主角，它的存在讲告知Spring容器需要为当前对象注入那些依赖对象。而@Component则是配合在配置文件中classpath-scanning功能使用的。在Spring的配置文件中增加一个“触发器”，使用@Autowired和@Component标注的类就能获得依赖对象的注入了。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Spring的配置文件中*<context:component-scan/>* 会到指定的包下面扫描标注有@Component的类，如果找到，则将它们添加到容器进行管理，并根据它们所标注的@Autowired为这些类注入符合条件的依赖对象。