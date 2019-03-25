## 1.8 Container Extension Points
通常，一个应用的开发者不需要继承ApplicationContext的实现类。相反，通过嵌入特殊接口的实现类，Spring IoC容器可以得到扩展。
这一章节将介绍这些可集成接口。

### 1.8.1 Customizing Beans by Using a `BeanPostProcessor`
`BeanPostProcessor`接口定义回调方法，通过实现这些回调方法，你可以提供自己的实例化逻辑，依赖解析逻辑等等。而且，如果你想要在容器完成实例化，配置和初始化一个bean
之后自定义一些逻辑处理，你可以嵌入一个或多个BeanPostProcessor的实现类。

你可以配置多个BeanPostProcessor实例，然后通过设置order属性来控制这些BeanPostProcessor实例的执行顺序。而使用order属性必须使BeanPostProcessor继承Order接口。
> `BeanPostProcessor`实例操作bean实例。Spring IoC 容器实例化一个bean实例，然后`BeanPostProcessor`实例对这些
> bean实例进行处理。

`org.springframework.beans.factory.config.BeanPostProcessor`接口由两个回调接口组成。当一个类在Spring IoC 容器中被注册为一个post-processor后置处理
器时，对于该Spring IoC 容器创建的每一个bean实例，Spring IoC容器在初始化方法(例如InitializingBean#afterPropertiesSet()或者生命的额init-method)调用之前以
及bean初始化结束之后，Spring IoC容器都会对这些后置处理器(的回调方法)进行调用。？Bean后置处理器通常会检查回调接口，后者用代理将一个bean包装起来。一些提供Spring
 AOP功能的基础类就是以Bean Post-Processor原理实现的，以提供代理包装逻辑。

`ApplicationContext`会自动检测在配置元数据中那些实现`BeanPostProcessor`接口的bean。`ApplicationContext`应用上
下文将这些bean注册为后置处理器以便于之后调用它们。Bean后置处理器在Spring IoC容器的部署方式和其他bean相同。
值得注意的是，当我们在Java配置类中使用@Bean factory method工厂方法声明一个BeanPostProcessor处理器时，这个工厂方
法的返回值应该是`BeanPostProcessor`接口实现类自身或者`BeanPostProcessor`接口，以清楚的指明该类是bean post processor的特性。由于BeanPostProcessor需要
被尽早的实例化以便于在应用到上下文中其它bean的初始化的过程中，因此`BeanPostProcessor`类型检测至关重要。
> 以代码的方式注册BeanPostProcessor实例
> 尽管BeanPostProcessor注册的以通过ApplicationContext自动检测发现的方式最为推荐，但是你也可以通过代码的方式调用
> `ConfigurableBeanFactory.addbeanPostProcessor`方法进行注册。以这种方法注册的BeanPostProcessor并不遵循
> `Ordered`接口。其注册的顺序就是其执行的顺序。以这种方式注册的BeanPostProcessor其执行顺序优先于由
> ApplicationContext自动发现并注册的BeanPostProcessor。

### 1.8.2 Customing Configuration Metadata with a BeanFactoryPostProcessor(通过BeanFactoryPostProcessor自定义配置元数据)
本小节讨论org.springframework.beans.factory.config.BeanFactoryPostProcessor。这个接口的语义与BeanPostProcessor非常相似，但又有一点很主要的不同之处：
BeanFactoryPostProcessor的作用对象是bean定义数据(Bean Definition Metadata)。即，Spring IoC容器允许BeanFactoryPostProcessor读取bean的定义配置并在
spring ioc容器实例化bean之前改变这些bean的定义配置。

我们可以配置多个BeanFactoryPostProcessor实例，通过设置order属性，你可设置这些BeanFactoryPostProcessor的运行顺序。然而，只有让BeanFactoryPostProcessor
实现Ordered接口才可以设置BeanFactoryPostProcessor的order属性。
> BeanFactoryPostProcessor只是改变bean定义配置，并未真正改变bean实例。真正的改变bean实例，你需要使用前面介绍的BeanPostProcessor。尽管在实现上通过
> BeanFactoryPostProcessor改变bean实例是可行的，但是这样做会引起bean过早实例化，违反了标准的bean生命周期，可能会使bean在实例化过程绕过bean post 
> processor的环节。

在ApplicationContext内声明的BeanFactory会被自动执行，以便将更改应用于定义容器的配置元数据。Spring自身也包含了许多预定义的bean factory post-processor，
例如`PropertyOverrideConfigurer`和`PropertyPlaceholderConfigurer`。你也可以自定义BeanFactoryPostProcessor，比如注册一个自定义的属性编辑器。
ApplicationContext能够自动发部署到上下文中的任何实现BeanFactoryPostProcessor接口的类。ApplicationContext将这些类当做Bean Factory Post-Processor处理
器使用。
> 和BeanPostProcessor一样，我们不希望BeanFactoryPostProcessor使用懒加载。因为如果没有没有被其他类引用，那BeanFactoryPostFactory就不会被实例化。因此，
> spring ioc容器不会将BeanFactoryPostProcessor标记为懒加载，并且Bean(Factory)PostProcessor会被急切的实例化，即使设置其为懒加载:
> `<bean defalut-lazy-init='true' />`.

#### Example:PropertyPlaceholderConfigurer
你可以使用`PropertyPlaceholderConfigurer`将bean definition配置的属性值外部化，即属性值可以从另外的Java Properties文件格式的文件中
读取并付给bean definition。这样在部署应用时，可对特定环境的属性进行定制化配置，比如数据库url和密码。

参考如下基于XML配置数据片段，对DataSource的属性定义使用了占位符:
``` xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
  <property name="location" value="classpath:com/something/jdbc.properties" />
</bean>

<bean id="dataSource" destroy-method="close" class="org.apache.commons.dbcp.BasicDataSource">
  <property name="driverClassName" value="${jdbc.driverClassName}" />
  <property name="url" value="${jdbc.url}" />
  <property name="username" value="${jdbc.username}" />
  <property name="password" value="${jdbc.password}" />
</bean>
```
该段代码展示了从一个外部的Properties文件(也就是没有在xml文件)配置属性值。在运行时，`PropertyPlaceholderConfigurer`被应用到元数据配置，对DataSource的的一些属性值进行替换。
要被替换的值被指定为${property-name}形式的占位符，它遵循Ant和log4j以及JSP EL样式。而真正的值来自于另一个标准的Java Properties文件。
``` java
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9092
jdbc.username=sa
jdbc.password=root
```
所以${jdbc.driverClassName}字符串在运行时被替换为值'sa',并且同样适用于与属性文件中的健匹配的其他占位符的值。`PropertyPlaceholderConfigurer`会检查bean定义的大多数属性
和属性中的占位符(嵌套使用占位符)。
在Spring2.5版本之后，我们可以使用一个专用的配置元素来配置属性占位符。如果有多个外部文件，则以逗号分隔:
`<context:property-placeholder location="classpath:com/something/jdbc.properties" />`
`PropertyPlaceholderConfigurer`不仅可以查找你指定的properties文件中的值。默认情况下，如果它在指定的properties文件中不能找到一个属性值，它还会查找Java系统的属性。
通过systemPropertiesMode可以配置这种占位符查找策略，策略可以选择以下三种中的一个:
1. `never` 0:never check system properties
2. `fallback` 1:check system properties if not resolvable in the specified properties files.This is the default.
3. `override` 2:check system properties first,before trying the specified properties files.This lets system properties override any other property source.

### 1.8.3 Customizing Instantiation Logic With a `FactoryBean`
我们可以实现org.springframework.beans.factory.FactoryBean接口来为Object创建自己的工厂方法。
`FactoryBean`接口是Spring IoC容器的实例化逻辑的一个嵌入点。如果有需要很复杂的实例化逻辑以代码的形式比在XML中配置更好的实现，就需要创建FactoryBean，在实现类中写入自己的
实例化逻辑，然后将自定义FactoryBean嵌入到Spring IoC容器中。
`FactoryBean`接口提供了三个方法:
- Object getObject():返回一个由该工厂方法创建的实例对象。该实例对象能否可以被共享，取决于该工厂方法返回的是singleton还是prototype类型的实例。
- boolean isSingleton():如果该工厂方法返回的是单例实例则返回`true`否则返回`false`。
- Class getObjectType():返回由getObject()方法返回的实例的类型(object type),如果事先不知道就返回null。

`FactoryBean`接口和概念用于SpringFramework中的许多位置。`FactoryBean`接口的50多个实现随Spring一起提供。
当你向Spring IoC 容器请求FactoryBean实例而不是由FactoryBean生产的实例，那么当我们调用ApplicationContext.getBean(beanId)时，需要给beanId添加'&'前缀。
所以，对于一个给定的id为'myBean'的FactoryBean实例，调用`getBean("myBean")`返回的是由FactoryBean生产的对象实例，调用`getBean("&myBean")`返回的是FactoryBean实例自身。

### 参考
[Spring IoC结构](https://juejin.im/entry/5a3f6909f265da43163d47f2)
[BeanFactory 和 FactoryBean](https://www.cnblogs.com/aspirant/p/9082858.html)
