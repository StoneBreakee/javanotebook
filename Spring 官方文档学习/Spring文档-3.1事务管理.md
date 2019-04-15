## 3.1 Spring Data Access
### 3.1.1 Spring Transaction Manager
每个应用都需要对数据做持久化操作，在持久化操作时需要连接数据库。一般都会将数据库连接作为一种资源进行管理，  
将这些资源组成资源池进行管理。其中每一个数据库连接内所做的操作都会涉及到数据库事务。Spring为此(数据库事务)  
进行了统一管理，使用PlatformTransactionManager接口。在每一个连接内做的事务操作由Spring事务进行创建，分配，
配置，执行，结束(回滚)。

在Spring声明式事务学习的过程中，需要用到面向切面(AOP)的知识，所以需要学习AOP  Aspect Oriented Programming with Spring
## 3.2 Aspect Oriented Programming with Spring
`The key unit of modularity in OOP is the class,whereas in AOP the unit of modularity is the aspect.`
面向对象中模块化的关键单元是class，而在AOP中模块化的关键单元是切面。
```
AOP is used in the Spring Framework to:
Provide declarative enterprise services. The most important such service is declarative transaction management.
Let users implement custom aspects, complementing their use of OOP with AOP.
```
AOP 在Spring Framework中的应用体现在:
1. 提供声明式企业级服务。这类服务中的典型代表是**声明式事务管理**.
2. 赋予用户实现自定义切面，利用AOP来协助补充OOP.

AOP 的重要概念
- Aspect(切面)：将关注点模块化，例如事物管理。在spring事物管理中，将各个与数据库交互的过程所涉及到的事物  
由原来的分散到各个方法中去自己维护改为由spring将其从各个方法中抽取出来，统一管理，通过aop功能，应用到原  
来的方法里。  
- Join Point(连接点)：程序执行的某个点，是指程序执行到某个函数或者某个异常处理，那么这个函数就是连接点。  
一个程序有很多个函数也就有很多个连接点，但并不是所有的连接点都会被Spring进行增强。  
- Advice(增强)：由切面在某些特殊的连接点上执行的特殊代码逻辑。增强的类型包括:aroud,before和after。  
许多AOP框架(包括Spring)将增强建模为拦截器，并且在连接点周围维护一个拦截链。增强=编写的增强逻辑+执行时机，  
执行时机就是around,before和after。
- Pointcut(切点)：A predicate that matches join points.用于匹配连接点的谓语表达式，通过该谓语表达  
式匹配到相应的一个或多个连接点。增强与切点表达式相关联，这样增强就能关联到连接点，根据增强自身的定义，在  
连接点执行的相应时机执行编写的增强逻辑。  
```
The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
```
由切点表达式匹配到连接点的概念是AOP的核心思想，Spring默认使用AspectJ切点表达式。
- Introduce(引入):为某种类型接口的Bean引入一个新的的方法或成员变量。Spring AOP允许我们引入一个接口和  
  一个实现类来应用到被增强的对象上。学习链接：[Introduce](https://www.cnblogs.com/lcngu/p/6346777.html)
- Target Object(目标对象):被一个或多个切面增强的对象.由于Spring AOP采用的是运行时代理，这个对象通  
  常是被增强后的代理。
- AOP Proxy(AOP代理):由AOP框架创建的对象，用于实现切面契约(在连接点上的增强执行等等)。在Spring  
  Framework，一个AOP代理可以是JDK动态接口代理也可以是CGLIB子类代理。
- Weaving(织入)：是一个过程，将切面(连接点+切点+增强)和应用中其他的Bean关联在一起生成一个增强后的对象。
  这种过程可以在编译期，加载期和运行期执行。Spring AOP，类似于其他纯Java AOP框架，在运行时执行织入。

在Spring的增强的几种增强类型中，围绕增强是最强有力的一种，其他的类型有前置增强，后置增强(又细分为：正常返回  
异常返回)。