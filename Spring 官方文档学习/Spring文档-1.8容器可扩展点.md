## 1.8 Container Extension Points
通常，一个应用的开发者不需要继承ApplicationContext的实现类。相反，通过嵌入特殊接口的实现类，Spring IoC容器可以得到扩展。
这一章节将介绍这些可集成接口。

### 1.8.1 Customizing Beans by Using a `BeanPostProcessor`
`BeanPostProcessor`接口定义回调方法，通过实现这些回调方法，你可以提供自己的实例化逻辑，依赖解析逻辑等等。而且，如果你想要
在容器完成实例化，配置和初始化一个bean之后自定义一些逻辑处理，你可以嵌入一个或多个BeanPostProcessor的实现类。

你可以配置多个BeanPostProcessor实例，然后通过设置order属性来控制这些BeanPostProcessor实例的执行顺序。而使用order属性必须
使BeanPostProcessor继承Order接口。