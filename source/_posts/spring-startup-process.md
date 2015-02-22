title: spring容器启动处理
date: 2015-02-22 18:03:51
categories: spring
tags:
---

Spring容器启动开始，首先会通过某种途径加载配置元数据，除了代码方式比较直接，在大部分情况下，容器需要依赖某些工具类（BeanDefinitionReader）对加载配置元数据进行解析和分析，并将分析后的信息编组为对应的BeanDefinition,最后把这些保存了Bean定义必要信息的BeanDefinition,注册到相应的BeanDefinitionRegistry。看下图：<!--more-->

**容器启动阶段类图**

![](/img/container startup.png)


**其它后处理类图**

![](/img/beanfactoryprocess.png)