## 1.8 Container Extension Points
通常，一个应用的开发者不需要继承ApplicationContext的实现类。相反，通过嵌入特殊接口的实现类，Spring IoC容器可以得到扩展。
这一章节将介绍这些可集成接口。

### 1.8.1 Customizing Beans by Using a `BeanPostProcessor`
`BeanPostProcessor`接口定义回调方法，通过实现这些回调方法，你可以提供自己的实例化逻辑，依赖解析逻辑等等。而且，如果你想要在容器完成实例化，配置和初始化一个bean之后自定义一些逻辑处理，你可以嵌入一个或多个BeanPostProcessor的实现类。

你可以配置多个BeanPostProcessor实例，然后通过设置order属性来控制这些BeanPostProcessor实例的执行顺序。而使用order属性必须使BeanPostProcessor继承Order接口。
> `BeanPostProcessor`实例操作bean实例。Spring IoC 容器实例化一个bean实例，然后`BeanPostProcessor`实例对这些
> bean实例进行处理。

`org.springframework.beans.factory.config.BeanPostProcessor`接口由两个回调接口组成。当一个类在Spring IoC 容器中被注册为一个post-processor后置处理器时，对于该Spring IoC 容器创建的每一个bean实例，Spring IoC容器在初始化方法(例如InitializingBean#afterPropertiesSet()或者生命的额init-method)调用之前以及bean初始化结束之后，Spring IoC容器都会对这些后置处理器(的回调方法)进行调用。？Bean后置处理器通常会检查回调接口，后者用代理将一个bean包装起来。一些提供Spring AOP功能的基础类就是以Bean Post-Processor原理实现的，以提供代理包装逻辑。

`ApplicationContext`会自动检测在配置元数据中那些实现`BeanPostProcessor`接口的bean。`ApplicationContext`应用上
下文将这些bean注册为后置处理器以便于之后调用它们。Bean后置处理器在Spring IoC容器的部署方式和其他bean相同。
值得注意的是，当我们在Java配置类中使用@Bean factory method工厂方法声明一个BeanPostProcessor处理器时，这个工厂方
法的返回值应该是`BeanPostProcessor`接口实现类自身或者`BeanPostProcessor`接口，以清楚的指明该类是bean post processor的特性。由于BeanPostProcessor需要被尽早的实例化以便于在应用到上下文中其它bean的初始化的过程中，因此`BeanPostProcessor`类型检测至关重要。
> 以代码的方式注册BeanPostProcessor实例
> 尽管BeanPostProcessor注册的以通过ApplicationContext自动检测发现的方式最为推荐，但是你也可以通过代码的方式调用
> `ConfigurableBeanFactory.addbeanPostProcessor`方法进行注册。以这种方法注册的BeanPostProcessor并不遵循
> `Ordered`接口。其注册的顺序就是其执行的顺序。以这种方式注册的BeanPostProcessor其执行顺序优先于由
> ApplicationContext自动发现并注册的BeanPostProcessor。

### 1.8.2 Customing Configuration Metadata with a BeanFactoryPostProcessor(通过BeanFactoryPostProcessor自定义配置元数据)
