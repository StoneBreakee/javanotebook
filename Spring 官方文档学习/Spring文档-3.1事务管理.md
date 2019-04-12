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
