title: Jetty介绍
date: 2015-06-16 16:47:45
categories: jetty
tags: [jetty]
---

## Jetty是什么

Jetty是一个开源项目，提供了HTTP server,HTTP client和Servletr容器。

1. 使用方面：去官网下载jetty,在使用时可以做修改操作，如修改端口，调整日记级别和像其他Servlet容器特征如JNDI,JMX,会话管理。
2. 扩展使用：提供深度覆盖的特定功能，比如我们的高可扩展性的异步客户端，Servlet代理配置，Jetty Maven插件，使用Jetty作为一个嵌入Web服务器。

## Jetty版本选择

|Version	|Year	|Home	    |JVM	|Protocols	                                        |Servlet |JSP	|Status      |
|-----------|-------|-----------|-------|---------------------------------------------------|--------|------|------------|
|9.3	    |2014	|Eclipse	|1.7	|HTTP/1.1, HTTP/2 RFC2616,javax.websocket, SPDY v3	|3.1	 |2.3	|Experimental|
|9.2	    |2014	|Eclipse	|1.7	|HTTP/1.1 RFC2616,javax.websocket, SPDY v3	        |3.1	 |2.3	|Stable      |
|8	        |2009-	|Eclipse/Codehaus	|1.6	|HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3	|3.0	|2.2	|Mature  |
|7	|2008-	|Eclipse/Codehaus	|1.5	|HTTP/1.1 RFC2616, WebSocket RFC 6455, SPDY v3	|2.5	|2.1	|Mature|
|6	|2006-2010	|Codehaus	    |1.4-1.5	|HTTP/1.1 RFC2616	|2.5	|2.0	|Venerable|
|5	|2003-2009	|Sourceforge	|1.2-1.5	|HTTP/1.1 RFC2616	|2.4	|2.0	|Deprecated|
|4	|2001-2006	|Sourceforge	|1.2, J2ME	|HTTP/1.1 RFC2616	|2.3	|1.2	|Ancient|
|3	|1999-2002	|Sourceforge	|1.2	|HTTP/1.1 RFC2068	|2.2	|1.1	|Fossilized|
|2	|1998-2000	|Mortbay	    |1.1	|HTTP/1.0 RFC1945	|2.1	|1.0	|Legendary
|1	|1995-1998	|Mortbay	    |1.0	|HTTP/1.0 RFC1945	|-	|-	|Mythical|


## Jetty and Java EE Web Profile

Jetty实现Java EE规范方面，主要是Servlet规范。最近的Java EE 平台发布对Web Profile做了介绍，也认识到了许多的开发人员只使用Java EE平台技术中的一部分。

Jetty本身没有使用所有的Web Profile技术，Jetty的架构是这样的，你可以通过第三方插件的方式实现产生一个容器定制化具体的需求。  



### JavaEE7 Web Profile

|JSR	 |Name	                 |Included with jetty-9.1.x	|Pluggable                             |
|--------|-----------------------|--------------------------|--------------------------------------|
|JSR 340	|Servlet Specification API 3.1	|Yes	 |              |
|JSR 344	|Java Server Faces 2.2 (JSF)	|No	|Yes, Mojarra or MyFaces|
|JSR 245 / JSR 341	|Java Server Pages 2.3/Java Expression Language 3.0 (JSP/EL)	|Yes	|Yes|
|JSR 52	|Java Standard Tag Library 1.2 (JSTL)	|Yes	|Yes|
|JSR 45	|Debugging Support for Other Languages 1.0	|Yes (via JSP)	|Yes (via JSP)|
|JSR 346	|Contexts and Dependency Injection for the JavaEE Platform 1.1 (Web Beans)	|No	|Yes, Weld|
|JSR 330	|Dependency Injection for Java 1.0	| No	| Yes as part of a CDI implementation, Weld|
|JSR 316	|Managed Beans 1.0	|No	 |Yes, as part of another technology|
|JSR 345	|Enterprise JavaBeans 3.2 |Lite	|No|	 
|JSR 338	|Java Persistance 2.1 (JPA)	|No|	Yes, eg Hibernate|
|JSR 250	|Common Annotations for the Java Platform 1.2	|Yes	|Partially (for non-core Servlet Spec annotations)
|JSR 907	|Java Transaction API 1.2 (JTA)	|Yes|	Yes|
|JSR 349	|Bean Validation 1.1	|No|	Yes as part of another technology eg JSF, or a stand-alone implementation such as Hiberate Validator|
|JSR 339	|Java API for RESTful Web Services 2.0 (JAX-RS)	|No |	|
|JSR 356	|Java API for Websocket 1.0	|Yes	| No |
|JSR 353	|Java API for JSON Processing 1.0 (JSON-P)	No|	|Yes, eg JSON-P reference implementation|
|JSR 318	|Interceptors 1.2	|No	 |Yes as part of a CDI implementation|






### Jetty EE 6 Web Profile

|JSR	    |Name	            |Included with jetty-9.0.x	|Pluggable      |
|-----------|-------------------|---------------------------|---------------|
|JSR 315	|Servlet Specification API 3.0	|Yes|                           |
|JSR 314	|JavaServer Faces 2.0 (JSF)	|No|	Yes, for example, Mojarra or MyFaces|
|JSR 245	|JavaServer Pages 2.2/Java Expression Language 2.2 (JSP/EL)	|Yes |	Yes|
|JSR 52	    |Java Standard Tag Library 1.2 (JSTL)	|Yes	|Yes|
|JSR 45	    |Debugging Support for Other Languages 1.0	|Yes (via JSP)	|Yes (via JSP)
|JSR 299	|Contexts and Dependency Injection for the Java EE Platform 1.0 (Web Beans)	| No	|Yes, Weld or OpenWebBeans
|JSR 330	|Dependency Injection for Java 1.0	|No|	Yes as part of a CDI implementation, Weld
|JSR 316	|Managed Beans 1.0	|No	|Yes, as part of another technology.
|JSR 318	|Enterprise JavaBeans 3.1	|No|	Yes, OpenEJB
|JSR 317	|Java Persistance 2.0 (JPA)	|No|	Yes, Hibernate
|JSR 250	|Common Annotations for the Java Platform	|Yes|	Partially (for non-core Servlet Spec annotations)
|JSR 907	|Java Transaction API (JTA)|	Yes|	Implementations are pluggable, such as Atomikos, JOTM, Jencks (Geronimo Transaction Manager)
|JSR 303	|Bean Validation 1.0	|No|	Yes as part of another technology (JSF), or a stand-alone implementation such as Hiberate Validator
