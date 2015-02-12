title: mybatis映射配置代理模式的使用
date: 2015-02-05 10:46:02
categories: mybatis
tags: [动态代理,mybatis]
---


## 1.mybatis映射简述
在使用mybatis过程中，一直有一个疑惑，在定义的DAO层接口并没有实现，接口类是怎么与映射文件进行连接的。接口名称与映射文件必须保持一直，为什么一定要这样做呢。一个接口类怎么做到与映射文件进行连接并执行sql，查看源代码中发现使用jdk动态代理，下面以一个简单的实例来说明一下流程。
<!--more-->
**接口：**  

     public interface NewsMapper {
           public String getNews(long id);

       }

**代理类：**

    public class MapperProxy implements InvocationHandler {

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		
		return "hello news";
	}
    }

**测试类：**

    public class Test {

	public static void main(String[] args) {

		NewsMapper mapper = (NewsMapper) Proxy.newProxyInstance(NewsMapper.class.getClassLoader(),
				new Class[] { NewsMapper.class }, new MapperProxy());

		String s = mapper.getNews(22);
		System.out.println(s);
	}
    }

从上面的例子中可以看出，映射接口并没有实现类，所以代理类中method 没有调用。这样做到了在接口没有任何实现类的情况获取返回值。大体上就是mybatis中接口与xml的关系。


## mybatis映射文件原理分析

**使用代码**

    InputStream  inputStream = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();
	NewsMapper newsMapper = sqlSession.getMapper(NewsMapper.class);

**类图**

![](/img/mybatis-mapper-proxy-flow.png)


**具体类说明**


SqlSessionFactoryBuilder：SqlSessionFactory 实例创建

DefaultSqlSession：SqlSession 的子类，定义数据库操作。

XMLConfigBuilder：读取mybatis-config.xml并解析，将将解析的内容存到配置类中

Configuration：用来存储操作mybatis-config.xml中的内容

MapperRegistry：处理与保持xml中Mapper对应的MapperProxyFactory。

MapperProxyFactory：创建mapper的接口的伪实例

MapperProxy：mapper接口代理类，并在invoke方法内返回MapperMethod的执行结果 

MapperMethod：此类用于将AccountSecurityMaster.xml中的insert,update,delete,select配置直接映射成jdbc的statement操作。