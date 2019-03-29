## 4.ApplicationContext介绍
### 4.1 WebApplicationContext介绍
WebApplicationContext是专门为Web应用准备的，它允许从相对于Web根目录的路径中装载配置文件  
完成初始化工作。从WebApplicationContext中可以获取ServletContext的引用，而  
`WebApplicationContext`将作为属性防止到ServletContext中，以便Web应用可以访问到Spring  
上下文。
``` java
/**
* This interface add a getServletContext() method to the generic Application Context.
* 该接口在通用ApplicationContext的基础上添加了getServletContext()方法。
*/
public interface WebApplicationContext{

  /**
   * 上下文启动时，WebApplicationContext即以此为键放置在ServletContext的属性列表中  
   * 可以通过如下语句从ServletContext中获取实例:
   * servletContext.getAttribute(WebApplicatonContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
  */
  String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = 
      WebApplicationContext.class.getName() + ".ROOT";

  /**
   * return the standard Servlet API ServletContext for this application.
   * 给当前应用返回标准的Servlet API中的ServletContext。
   * 每个Web应用在Web容器中对应一个ServletContext，该ServletContext需要与当前应用下的  
   * Spring上下文进行交互，以启动Spring上下文进行交互。
  */
  ServletContext getServletContext();
}
```
ConfigurableWebApplicationContext扩展了WebApplicationContext，它允许通过配置的方式实  
例化WebApplicationContext，同时定义了两个重要的方法:
- setServletContext(ServletContext servletContext):为Spring设置Web应用上下文，以便  
  二者整合。结合下文可知：Web容器启动时启动了加载了ServletContext，启动Spring容器提供的Servlet  
  在实例化Spring Web上下文WebApplicationContext的过程中，将Spring上下文与ServletContext关  
  联在一起。
- setConfigLocations(String[] configLocations):设置Spring配置文件地址。用户可以使用带资源  
  类型前缀的地址，如classpath:com/beans.xml等。

### 4.2 WebApplicationContext 初始化
WebApplicationContext的初始化方式和BeanFactory和ApplicationContext有所区别，因为WebAppl-  
icationContext在启动时需要ServletContext实例，也就是说，它必须在拥有Web容器的前提下才能完成  
启动工作。？可以在web.xml中配置自启动的Servlet或定义Web容器监听器？ -> web.xml文件的作用？  
(web.xml是Web容器启动时为应用生成的ServletContext?)  
Spring分别提供了用于启动WebApplicationContext的Servlet和Web容器监听器:  
- org.springframework.web.context.ContextLoaderServlet
- org.springframework.web.context.ContextLoaderListener
两者内部都实现了启动WebApplicationContext实例的逻辑，只要根据Web容器的具体情况二者选一，并在  
web.xml中完成配置即可。