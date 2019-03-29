## 1.9 Annotation-based Container Configuration
基于注释的配置提供了XML配置的替代方案，该依赖配置依赖于字节码元数据来链接组建而不是通过  
XML中的`<bean>`角括号声明。开发人员在相关类，方法和字段声明上使用注解将Bean配置移动到  
组建类自身上。但是使用BeanPostProcessor来扩展Spring IoC容器是两种配置方式的共同点。  

- 在Spring2.0中引入了@Required注解，其作用是强制装配所需属性。  
- 在Spring2.5中引入了@Autowired注解，其作用与@Reuqired注解装配属性类似。但是本质上  
  @Autowired注解提供了更细致粒度的控制和更广泛的适用性。
- Spring2.5也增加了对JSR-250注解的支持，例如@PostConstruct和@PreDestroy注解。
- Spring3.0增加了对`JSR-330`(基于Java原生的依赖注入)的支持，这些注解在`javax.inject`  
  包中，注解有@Named,@Inject等。
  
> 注解注入的执行顺序优先于XML注入。因此，XML配置会覆盖注解配置的属性配置。

像`PropertyPlaceHolderConfigurer`一样，在使用注解配置时需要向容器注册一系列的`BeanPostProcessor`  
来进行处理，我们可以在`<bean>`中定义这些处理类，也可以通过`<context:annotation-config />`配置  
来进行隐式的声明。
> 这些隐式被注册的后置处理器包括
> `AutowiredAnnotationBeanPostProcessor`,`CommonAnnotationBeanPostProcessor`
> `PersistenceAnnotationBeanPostProcessor`和`RequiredAnnotationBeanPostProcessor`.

### 1.9.1 @Required注解
@Required注解可用于bean property的setter方法注入，代码如下:
``` java
public class SimpleMovieLister{
  private MovieFinder movieFinder;

  @Required
  public void setMovieFinder(MovieFinder movieFinder){
    this.movieFinder = movieFinder;
  }
  // ...
}
```
此注解表示必须在配置时填充受影响的bean属性，可以通过在beandefinition中显式的指定property的  
值也可以通过自动装配。如果受影响[即被@Required注解的]的bean属性没有被填充，Spring IoC 容器  
就会抛出异常,从而防止程序运行之后抛出`NullPointerException`.建议对通过类自身的方法对属性进行  
判断,比如使用init-method等。
> @Required注解在SpringFramework5.1中被正式废弃。建议使用constructor注入来确保配置强依赖的属性

### 1.9.2 Using @Autowired
