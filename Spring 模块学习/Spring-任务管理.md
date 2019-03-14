### Spring Schedule 任务管理 定时任务

Spring 定时任务也可以叫做任务管理，有两种实现方式
- @Scheduled
- Quartz

#### 1.Scheduled
基于`@Scheduled`注解的定时任务  
1.向系统注册使用任务管理
在主类上添加`@EnableScheduling`注解，或则在配置文件中添加`<task:annotation-driven/>`  
2.在需要定时执行的方法上添加`@Scheduled`注解，该注解有多个参数可以接收，其主要代码注释如下：  
``` java
An annotation that marks a method to be scheduled. Exactly one of
the cron(), fixedDelay(), or fixedRate()
attributes must be specified.

该注解用于标记一个方法，是该方法被任务管理调度，成为定时任务。
使用该注解时 cron，fixedDelay或者fixedRate中的一个应该被准确指明。

The annotated method must expect no arguments. It will typically have
a {@code void} return type; if not, the returned value will be ignored
when called through the scheduler.

被注解的方法不需要有入参。通常，该方法的返回类型应该是void；如果其有返回类型，那么该方  
法的返回值在该方法被任务管理器调用的时候忽略调。

Processing of {@code @Scheduled} annotations is performed by
registering a {@link ScheduledAnnotationBeanPostProcessor}. This can be
done manually or, more conveniently, through the {@code <task:annotation-driven/>}
element or @{@link EnableScheduling} annotation.

@Scheduled 注解的处理是由ScheduledAnnotationBeanPostProcessor完成的，我们不必手动处理。可以通过<task:annotation-driven/>配置  
或者@EnableScheduling来自动完成。
```

所有被`@Scheduled`注解的方法，首先被系统处理放入定时任务管理队列中，然后由 **单线程逐一执行**，最初始执行顺序cronTask > ...  
如果中间某个任务执行过长而耽误了后面的任务执行

- postProcessorAfterInitialiation  
Spring IoC容器创建之后，会扫描ApplicationContext中的所有bean配置，然后根据bean配置对bean进行组装，在组装过程中，如果Spring IoC容
器中存在BeanPostProcessor组建，则会在装配Bean的过程中，调用BeanPostProcessor的postProcessorAfterInitial方法中对bean的逻辑进行扩展。
当Spring IoC容器创建Bean的过程中，对原始Class进行实例化之后，将该class实例封装到BeanDefinition中，在BeanDefinition中对
原始实例进行多层增强(aop)和代理(proxy)，在此封装过程中，对原始类的代码中存在的注解等进行反射等以填充BeanDefinition和其它逻辑

- Spring @Scheduled注解  
该注解由ScheduledAnnotationBeanPostProcessor类负责。首先，由Spring IoC容器在对类进行Bean实例化时，当碰到@Scheduled注解的方法
交由ScheduledAnnotationBeanPostProcessor的postProcessorAfterInitialization方法进行处理，之后再由afterProperties方法进行
任务管理的配置。

``` java
ScheduledAnnotationBeanPostProcessor
|- scheduler:在registrar将被@Scheduled注解的方法转换为task之后，执行task任务
|- registrar:把被@Scheduled注解的方法转换为task任务，并集中管理，然后交由scheduler(TaskScheduler)执行。
|- setScheduler(Object scheduler):该方法负责装配scheduler属性，如果没有显式
|-                                  使用该setter方法注入，Spring IoC容器在装配该属性时逻辑如下：
|-                 第一步：在ApplicationContext中查找一个类型为TaskScheduler的bean，或者beanName为taskScheduler的bean进行装配
|-                 第二步：在如没有查找到，则查找ScheduledExecutorService。
|-                 第三步：如果都没有查找到，则由registrar创建一个本地单线程的scheduler执行task。
```

- ScheduledAnnotationBeanPostProcessor.java  
BeanPostProcessor接口在Bean的生命周期中负责在Bean完成实例化之后进行postProcessorAfterInitialized方法回调,目的在与对bean的某些属性进行包装(wrapper)  
ScheduledAnnotationBeanPostProcessor实现了BeanPostProcessor接口，用于在Bean实例化之后，进行回调，将原始class实例中含有@Scheduled注解的方法包装为由Spring 容器可以定时调度的Task，并交由系统TaskScheduler调度执行
<pre><code>
/**
 * 1.根据bean实例获取原始class类实例对象的class类信息
 * 2.检查该class中是否有被@Scheduled注解的方法
 * 3.如果有，则将该方法转换为Task任务，交由TaskScheduler调度执行
 * */
@Override
public Object postProcessAfterInitialization(final Object bean, String beanName) {
    Class<?> targetClass = AopProxyUtils.ultimateTargetClass(bean);
    if (!this.nonAnnotatedClasses.contains(targetClass)) {
        Map<Method, Set<Scheduled>> annotatedMethods = MethodIntrospector.selectMethods(targetClass,
                new MethodIntrospector.MetadataLookup<Set<Scheduled>>() {
                    @Override
                    public Set<Scheduled> inspect(Method method) {
                        Set<Scheduled> scheduledMethods = AnnotatedElementUtils.getMergedRepeatableAnnotations(
                                method, Scheduled.class, Schedules.class);
                        return (!scheduledMethods.isEmpty() ? scheduledMethods : null);
                    }
                });
        if (annotatedMethods.isEmpty()) {
            this.nonAnnotatedClasses.add(targetClass);
            if (logger.isTraceEnabled()) {
                logger.trace("No @Scheduled annotations found on bean class: " + bean.getClass());
            }
        } else {
            // Non-empty set of methods
            for (Map.Entry<Method, Set<Scheduled>> entry : annotatedMethods.entrySet()) {
                Method method = entry.getKey();
                for (Scheduled scheduled : entry.getValue()) {
                    processScheduled(scheduled, method, bean);
                }
            }
            if (logger.isDebugEnabled()) {
                logger.debug(annotatedMethods.size() + " @Scheduled methods processed on bean '" + beanName +
                        "': " + annotatedMethods);
            }
        }
    }
    return bean;
}
</code></pre>
<pre><code>
// TargetClassAware
// XXAware 感知接口目的是使Bean实例获取Spring 中的相关信息，
// 如BeanNameAware是为了让Bean实例获取该Bean实例在Spring 容器中的beanName(也即是id)
// 在定义Class类的时候，实现BeanNameAware的setBeanName(String beanName){this.beanName = beanName}方法，之后Spring系统会回调该方法以将该Bean的beanName传送给该bean实例
// TargetClassAware 可以获得该类的被代理的对象的Class类型
public static Class<?> ultimateTargetClass(Object candidate) {
    Assert.notNull(candidate, "Candidate object must not be null");
    Object current = candidate;
    Class<?> result = null;
    // 可能存在多层代理
    while (current instanceof TargetClassAware) {
        result = ((TargetClassAware) current).getTargetClass();
        current = getSingletonTarget(current);
    }
    if (result == null) {
        result = (AopUtils.isCglibProxy(candidate) ? candidate.getClass().getSuperclass() : candidate.getClass());
    }
    return result;
}
</code></pre>


Spring Bean Initialize 过程(乞丐版)
1. 根据配置文件或扫描工程文件获取组件Bean列表，并根据组件Bean事先定义好的配置,beanName和class属性，生成BeanDefinition实例
2. 根据class属性指向的类文件的binary name(全路径名com.xx.xx),使用反射机制生成Class<?>对象，再调用newInstance()方法，生成class类型的
   实例，用BeanWrapper将这个实例包装起来。(如果属性是根据构造器装配的，则调用有参构造函数,可能会引起 **循环依赖** ;否则，调用默认空参
   数的构造函数)
3. 在BeanWrapper将class对象实例化包装之后，接着调用populateBean(..)方法将属性填充
4. 在属性填充之后，调用invokeAwareMethods(..)，根据事先实现的**Aware接口，获取相应的系统属性
4. 获取系统的属性之后，调用invokeInitMethod(..)方法执行事先定义的initMethod方法
5. 在执行完initMethod方法之后，循环遍历Spring IoC容器中的BeanPostProcessor组件，并调用postProcessorAfterInitialization(..)方法
6. ...
7. 发布ApplicationContextRefresh()事件，实现监听事件的组件执行响应方法onApplicationEvent(...)
#### 2.Spring Quartz

应用场景
- 每半个小时查询用户是否有快到期的待处理业务

#### 3.参考链接  
[Cron 表达式 用法](https://www.cnblogs.com/mingyue1818/p/5764050.html)  
[Cron 表达式 在线生成](http://cron.qqe2.com/)  
[Spring Quartz 实现原理及示例](https://www.cnblogs.com/SpaceAnt/p/6354446.html)  
[Spring注解@Resource和@Autowired区别对比](https://www.cnblogs.com/think-in-java/p/5474740.html)  
[Spring @Scheduled task-annotation scheduler配置](https://blog.csdn.net/yx0628/article/details/80873774)