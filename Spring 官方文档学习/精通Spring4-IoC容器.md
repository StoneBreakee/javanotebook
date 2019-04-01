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

### 4.3 Bean的生命周期
Bean的生命周期由多个特定的生命阶段组成，每个生命阶段都开出了一扇门，允许外界由此门对Bean施加控制。  
下面分别对BeanFactory和ApplicationContext中Bean的生命周期进行分析。

##### 4.3.1 BeanFactory中Bean的生命周期
起点:[通过getBean()调用某一个Bean]  
经由:  
1.  **调用InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation()方法**  
2.  实例化
3.  **调用InstantiationAwareBeanPostProcessor的postProcessAfterInstantiation()方法**
4.  **调用InstantiationAwareBeanPostProcessor的postProcessPropertyValues()方法**
5.  设置属性值
6.  调用BeanNameAware的setBeanName()方法
7.  调用BeanFactoryAware的setBeanFactory()方法
8.  **调用BeanPostProcessor的postProcessBeforeInitialization()方法**
9.  调用InitializingBean的afterPropertiesSet()方法
10. 通过init-method属性配置的初始化方法
11. **调用BeanPostProcessor的postProcessAfterInitialization()方法**
12. singleton -> Spring缓存池中准备就绪的Bean;
    prototype -> 将准备就绪的Bean交给调用者
13. 容器销毁 -> 调用DisposableBean的destroy()方法
14. 通过destroy-method属性配置的销毁方法
    
>`BeanPostProcessor`在Spring框架中占重要地位，为容器提供对Bean进行后续加工处理的切入点，Spring 
> 容器所提供的各种"神奇功能"(如AOP，动态代理)都通过BeanPostProcessor实施。

Bean的生命周期从Spring容器着手实例化Bean开始，直到最终销毁Bean，其中经过了许多关键点，每个关键点都  
涉及特定的方法调用，可以将这些方法大致划分为4类:
1. Bean自身的方法:如调用Bean构造函数实例化Bean，调用Setter设置Bean的属性值及通过`<bean>`的init  
   -method和destory-method所指定的方法。
2. **Bean级生命周期接口方法**:如BeanNameAware，BeanFactoryAware，InitializingBean和Disp-  
   -osableBean，这些接口方法由Bean类直接实现。
3. **容器级生命周期接口方法**:在上述步骤中带有加粗的步骤是由
   `InstantiationAwareBeanPostProcessor`和`BeanPostProcessor`这两个接口实现的，一般称  
   它们的实现类“后处理器”。后处理器接口一般不由Bean本身实现，它们独立于Bean，它们的实现类以容器  
   附加装置的形式注册到Spring容器中，并通过接口反射为Spring容器扫描识别[这里涉及到`BeanFactory`  
   和Application的区别，Spring容器在BeanFactory的情况下需要手工注册，在ApplicationContext  
   场景下可实现自动扫描发现机制]。  
     