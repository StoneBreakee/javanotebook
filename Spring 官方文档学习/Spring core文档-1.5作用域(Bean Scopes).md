## 1.5 作用域

bean
一个典型的企业级应用不是由单个对象(用Spring的属于来说,是Bean)组成的。即使是最简单的应用也是由一些对象共同完成，以呈现最终用户所看到的一体的应用程序。

> Object vs Bean  
> - 在一般的Java应用中，比如Helloworld.java，HelloWorld是一个class类，可以被java虚拟机实例化为一个对象object  
> - 在Spring IoC容器汇中，比如HelloWorld.java，在Spring IoC容器对其加载和实例化的过程中，是以bean为单位的。普通的Java对象在Spring IoC容器中，被封装为一个BeanDefinition以描述一个完整的Spring bean对象。

### 1.4.1 Dependency Injection
依赖注入是一个过程，由此，objects仅仅通过构造函数，工厂方法参数或属性 **定义** 它的依赖(这些依赖在objects构造完成或者从工厂方法返回后被设置在objects 实例上)。接着，Spring IoC容器在创建bean对象的时候 **注入** 这些依赖。这个过程是bean自身的反转，通过反转 直接构造类或者服务定位模式 控制其依赖(名词)的实例化或者定位。

DI有两个主要的方式：基于构造函数的依赖注入和基于setter的依赖注入。  

- 基于构造函数的依赖注入

基于构造函数的依赖注入的完成由Spring IoC 容器调用带有多个参数的构造函数完成，每个参数代表一个依赖。调用工厂方法与调用构造函数类似。例如，SimpleMovieLister是一个普通的POJO，没有实现Spring IoC容器的任何特殊接口，基础类和注解。

``` java
public class SimpleMovieLister{
    
    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor that the spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder){
        this.movieFinder = movieFinder;
    }
}
```
SimpleMovieLister是一个普通的POJO，没有实现Spring IoC容器的任何特殊接口，基础类和注解。

- 基于Setter函数的依赖注入

基于Setter函数的依赖注入的是由Spring IoC 容器为了实例化bean调用一个无参构造构造方法(或工厂方法)之后调用bean的setter方法完成的。例如，SimpleMovieLister是一个普通的POJO，没有实现Spring IoC容器的任何特殊接口，基础类和注解。

``` java
public class SimpleMovieLister{
    
    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a setter method that the spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder){
        this.movieFinder = movieFinder;
    }
}
```
###### 使用哪种依赖:Constructor-based or setter-based DI?
constructor-based DI 可以用于必须使用的依赖对象，而setter-based DI用于可以选择的依赖对象。Spring Team通常建议使用构造器依赖注入，这样可以使你的application 组件成为不必再更改的对象，也确保了必要的依赖(名词)不为空。更近一步的讲，构造器注入组件总是返回给调用者一个完整实例化的bean组件。另外值得注意的是，如果构造函数中有太多参数，那么你的代码就不够友好，常常意味着这个bean组件所负责的功能太多，应该对其进行重构以分离功能点。

Spring IoC容器在容器被创建之后会对每一个bean的配置进行校验。而每个bean的properties属性直到该bean被创建的时候才进行装配。那些Scope为Singleton的beans和被设置为预实例化的beans会在容器创建随后创建。Scopes属性的定义是在Bean Scopes中设置的。其它的beans,则是在该bean需要的时候才被实例化。一个bean的创建隐藏着一系列beans的创建，比如该bean所依赖的bean和该bean所依赖的bean的依赖bean，都会被创建和分配。

### 1.4.4 懒加载初始化Beans

默认地，在Spring IoC 容器初始化过程中，ApplicationContext的实现包括了创建和配置所有作用域为singleton的bean。通常情况下，这种预初始化是需要的。你可以通过配置bean definition中属性以启动懒加载模式。被设置为懒加载的bean会告诉IoC容器只有在该bean被用到的时候，才将该bean实例化，而不是在Spring IoC 容器启动的时候实例化。  
然而，当一个singleton的bean的一个依赖是懒加载的bean时，ApplicationContext也会在实例化singleton beans的时候将lazy-initiating beans实例化。

### 1.4.5 自动装配协作者(Autowiring Collaborators)
在此，将Collaborater理解为beans的依赖，即bean需要它的依赖来共同完成某项工作。
> 区别与Spring的注解@Autowired,注解只是自动装配的一种使用。

该应用中，有不止一个bean，这些beans之间存在依赖关系，因此这些beans也可以理解为Collaboraters。Spring IoC容器可以自动识别并装配这些collaborater beans之间的依赖关系。通过识别ApplicationContext上下文中的所有bean组件，Spring可以为你自动识别bean中依赖所需要的collaborater bean实例。
> bean中的依赖(名词) --> 为这些依赖(dependencies)注入实例 --> 途径有constructor，setter方法 --> 手动完成  

- Autowiring 自动装配：对于某个bean，Spring IoC在实例化时识别其依赖(名词)，然后根据某种规则，自动发现其collaborator beans,然后自动将实例注入到其依赖(名词)中，整个过程称为自动装配。

- ApplicationContext是指：在Spring IoC 容器中存在很多beans，beans也都会有一个beanname，以XML配置为例，这些beans的元数据配置(class,依赖，scope等等)都是在XML中配置的，当Spring IoC 容器启动时回去读取这些配置然后根据配置实例化beans(预加载或懒加载)，而这些加载后的beans在容器中组成ApplicationContext。根据Spring IoC容器读取配置的方式又可以有Classpath...ApplicationContext,FileSystem...Application,如果是Web项目又可以有Web...ApplicationContext。

When using XML-based configuration metadata (see Dependency Injection), you can specify the autowire mode for a bean definition with the autowire attribute of the <bean/> element. 
当使用基于XML文件配置beans时，你可以为一个bean指定自动装配模式，仅仅通过配置文件中<bean>元素的一个autowire属性。  
` <bean id=... class=... autowire="byName|byType|constructor">`  
下面的表格描述Spring 自动装配的四种方式。

|Mode|Explanation|
|:---|:----------|
|no|默认情况下不开启自动装配功能，也是默认模式。Bean的依赖必须通过<bean>的ref元素指定。对于大型项目不建议改变默认模式，因为通过显示的指定collaborater bean使配置更清晰更容易控制。在某种程度上，它也能看成是系统的说明文档。|
|byName|根据属性名进行自动装配.Spring寻找与property 名字相同的bean name，然后将其装配到property上。例如，如果一个bean被设置为自动装配，并且该bean有一个属性(也可理解为依赖)，名字为master，且拥有一个setMaster(..)方法，spring就会为其寻找一个bean name为master的bean，然后将master bean装配到master property上|
|byType|如果在Spring IoC 容器中有且只有一个collaborater bean的class类型与bean 的property的class类型相同，才可以通过byType进行属性的自动装配.如果存在多个与property class类型相同的collaborater bean，就会抛出异常。如果没有匹配的collaborater bean，不会抛出异常，property也不会被装配|
|constructor|与byType类似，但是适用于使用constructor构造函数进行依赖注入的情况。也会抛出异常。|

byName，byType使用适用于基于setter方法进行依赖注入的方式的自动装配，constructor使用适用于基于constructor方法进行依赖注入的方式进行自动装配。而Spring引入的自动装配的注解@Autowired是基于byType实现的，其装配规则如下：

1. 首先，Autowired默认先按byType注入
2. 如果发现有多个匹配的collaborater bean，则又按照byName的方式比对

关于1.4.5小节的参考资料  
1. [自动注入的几种方式](https://www.cnblogs.com/ChrisMurphy/p/5064822.html)  
2. [@Autowired与自动装配的关系](http://swiftlet.net/archives/734)

### 1.4.6 [方法注入](https://spring.io/blog/2004/08/06/method-injection/)

参考资料：[方法注入](https://yq.aliyun.com/articles/73831)  
In most application scenarios, most beans in the container are singletons. When a singleton bean needs to collaborate with another singleton bean or a non-singleton bean needs to collaborate with another non-singleton bean, you typically handle the dependency by defining one bean as a property of the other. A problem arises when the bean lifecycles are different. Suppose singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container creates the singleton bean A only once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.

A solution is to forego some inversion of control. You can make bean A aware of the container by implementing the ApplicationContextAware interface, and by making a getBean("B") call to the container ask for (a typically new) bean B instance every time bean A needs it.
在大部分的应用场景中，大多数的bean在容器中的作用域都是单例的。当一个单例模式的bean需要依赖另一些单例模式的bean时(预加载或懒加载)或则非单例模式的bean依赖一些非单例模式的bean时，你可以向往常那样通过通过将bean装配到property上。但是，当两个bean的生命周期不相同时，就会出现异常。假设，一个singleton bean A需要一个non-singleton bean B(通常是原型bean，prototype bean)，A在其每个方法中都会用到B。Spring IoC 容器在生命周期内只会创建一次A，因此也只有一次装配property的机会。 **Spring IoC 容器不能给A每调用一次方法都生成一个bean B的新实例。**  
解决方案是放弃一些原有的控制反转的便利。你可以使A通过实现ApplicationContextAware接口以获取Spring IoC容器上下文的一些信息，然后通过上下文的getBean("B")调用容器查找并请求一个新的bean B实例以供A使用。例子如下。
> 原型Bean(Prototype Bean)是每次使用都要创建一个新的Bean？见下一小节1.5

``` java
// a class that use a stateful Command-Style class to perform some processing
package fiona.apple;

// spring API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {
    
    private ApplicationContext applicationContext;

    public Object process(Map commandState){
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }
    
    protected Command createCommand(){
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command",Command.class);
    }
    
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException{
        this.applicationContext = applicationContext;
    }
}
```

上面的这段代码还可以改进，不足之处是将业务逻辑代码和框架代码耦合在一起了。方法注入，不像constructor或setter的方式，可以将property作为后两者的参数，然后通过IoC容器进行装配。方法注入与Constructor和Setter相比，更类似与Getter，无需参数也能返回一个bean组件以供使用。方式注入，作为Spring IoC容器的高级特征，可以更简洁的解耦代码进行处理。
- Lookup Method Injection

查询式方法注入也是容器的一种功能，它可以重写基于容器管理的bean组件的方法并在容器中查找匹配的被命名的bean组件作为方法返回的结果。这种查询的主要目标是prototype bean。

1. 重写的方式：
Spring Framework通过使用CGLIB库的字节码生成来生成一个子类以完成方法的覆盖。
2. 被覆盖的方法在子类中由Spring IoC实现内部逻辑，最终返回需要的被命名好的bean。

需要被覆盖的方法是abstract的，因此该类和该方法都不能是final。具体例子如下。

``` java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager{
    public Object process(Object commandState){
        // grab a new instance of the approriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method
    protected abstract Command createCommand();
}
```

在使用端类(包含需要被注入的方法，比如CommandManager)，被注入的方法的签名如下：
`<public|protected> [abstract] <return-type> theMethodName(no-arguments);`

如果方法是abstract，那么动态生成的子类会实现该方法；否则，动态生成的子类会覆盖该方法。
``` XML
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand" />
</bean>

或者
@Lookup("myCommand")
protected abstract Command createCommand();

// byType 查找bean
@Lookup
protected abstract Command createCommand();
```
这样CommandManager每次调用createCommand()方法时，都会返回一个新的bean实例。