org.springframework.beans.factory.support.AbstractBeanFactory
该类还提供了模板方法供子类实现:getBeanDefinition,createBean.
过程为：根据beanName检索到BeanDefinition，根据检索到的BeanDefinition创建bean实例.
此类继承了DefaultSingletonBeanRegistry,DefaultSingletonBeanRegistry类对不同程度(正在创建，待创建，创建完成...)的单例bean进行分类缓存
``` java
@param typeCheckOnly 用于检查获取的bean是否只是为了类型检查而不是真正的使用
protected <T> T doGetBean(final String name, final Class<T> requireType,final Object[] args,boolean typeCheckOnly) throws BeansException {
  // 解析别名，如果name中包括'&',则去掉
  final String beanName = transformedBeanName(name);
  Object bean;

  // 检查单例缓存,该缓存缓存的是已经注册过的单例
  Object sharedInstance = getSingleton(beanName);
  if(sharedInstance != null && args == null){
    bean = getObjectForBeanInstance(sharedInstance,name,beanName,null);
  }
  else{
    // 检查该bean是否 正在 创建该bean:如果正在创建，我们假设发生了循环依赖
    if(isPrototypeCurrentlyInCreation(beanName)){
      throw new BeanCurrentlyInCreationException(beanName);
    }

    // 检查bean definition是否在父类BeanFactory中
    ... ...

    if(!typeCheckOnly){
      // 将该bean标记为已经被创建
      markBeanAsCreated(beanName);
    }
    // 在该bean标记为已经被创建后，开始创建该bean
    try{
      // 根据beanName获取该bean的BeanDefinition -> 由DefaultListableBeanFactory.getBeanDefinition(String beanName)获取BeanDefinition
      // 然后将自身BeanDefinition与父类BeanDefinition合并,并将合并后的RootBeanDefinition放入缓存中,方便下次使用
      final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      checkMergedBeanDefinition(mbd,beanName,args);

      // 确保该bean所依赖的bean均已被初始化
      String[] dependsOn = mbd.getDependsOn();
      if(dependsOn != null){
        for(String dep:dependsOn){
          if(isDependent(beanName,dep)){
            // 循环依赖
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
          }
          registerDependentBean(dep,beanName);
          try{
            getBean(dep);
          }catch(NoSuchBeanDefinitionException ex){
            throw new BeanCreationException(mbd.getResourceDescription(),beanName,"'"+beanName+"' depends on missing bean '" + dep + "'",ex);
          }
        }
      }

      // 创建bean实例
      if(mbd.isSingleton()){
        sharedInstance = getSingleton(beanName,new ObjectFactory<Object>(){
          @Override
          public Object getObject() throws BeansException{
            try{
              // AbstractBeanFactory的模板方法，具体逻辑实现由子类完成，可参见AbstractAutowiredCapableBeanFactory.createBean(...)
              return createBean(beanName,mbd,args);
            }catch(BeanException ex){
              destroySingleton(beanName);
              throw ex;
            }
          }
        });
        bean = getObjectForBeanInstance(sharedInstance,name,beanName,mbd);
      }
    }catch(){

    }
  }
}
```

org.springframework.beans.factory.support.AbstractAutowiredCapableBeanFactory
实现了创建bean方法的抽象bean工厂的超类(其它的抽象类没有实现创建bean的方法),并且具有RootBeanDefinition指定的所有功能.
同时实现了AutowiredCapableBeanFactory接口和AbstractBeanFactoryBean抽象类的createBean接口.
该类提供了bean创建，属性填充和初始化。
支持构造函数自动装配，属性填充[byName,byType]
该类列举的需要由子类实现的主要模板方法有`resolveDependency(DependencyDescriptor,String,Set,TypeConverter)`,用于根据类型(byType)实现自动装配。
``` java
// 该类的核心方法
// create a bean instance,populates the bean instance,applies post-processor等等
@Override
protected Object createBean(String beanName,RootBeanDefinition mbd,Object[] args) throws BeanCreationException{
  RootBeanDefinition mbdToUse = mbd;

  // 确保RootBeanDefinition中的beanClass已经被解析,也就是mbd中beanClass对象已经存在
  Class<?> resolvedClass = resolveBeanClass(mbd,beanName);
  // 对原有的RootBeanDefinition进行深度拷贝
  // 原因未明？
  mbdToUse = new RootBeanDefinition(mbd);
  mbdToUser.setBeanClass(resolvedClass);
  ...
  Object beanInstance = doCreateBean(beanName,mbdToUse,args);
  return beanInstance;
}

// 该方法完成具体的bean创建.bean创建之前的预处理Pre-creation processing已经处理.
// 区分默认bean实例化,使用工厂方法和自动装配构造函数
//    先查看BeanDefinition是否定义了工厂方法，如果是则使用2
//    然后根据构造函数区分使用自动装配构造函数还是使用默认bean实例化(无参构造函数)
protected Object doCreateBean(final String beanName,final RootBeanDefinition mbd,final Object[] args) throws BeanCreationException{
  // 实例化bean
  BeanWrapper instanceWrapper = null;
  if(mbd.isSingleton()){
    // 检查是否有缓存
    // 该缓存存储的是未完成的factoryBean  Map[beanName -> BeanWrapper]
    instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
  }
  if(instanceWrapper == null){
    instanceWrapper = createBeanInstance(beanName,mbd,args);
  }

  final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
  Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);
  mbd.resolvedTargetType = beanType;

  // 容器中注册的后置处理器(post-processors)对RootBeanDefinition进行操作
  ...
  // 注意：到此，bean的实例化已经完成,但是初始化工作尚未完成
  // 将已经实例化完成的bean缓存起来，以解决循环依赖的问题
  boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
  if(earlySingletonExposure){
    if(logger.isDebugEnabled()){
      logger.debug("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
    }
    // 待补充 DefaultSingletonBeanRegistry
    addSingletonFactory(beanName,new ObjectFacory<Object>(){
      @Override
      public Object getObject() throws BeansException{
        return getEarlyBeanReference(beanName,mbd,bean);
      }
    });

    // 初始化Bean实例
    Object exposedObject = bean;
    try{
      // 填充bean属性,通过BeanWrapper设置Property的值
      populateBean(beanName,mbd,instanceWrapper);
      if(exposedObject != null){
        // 初始化bean实例，应用工厂回调以及init方法和bean的后置处理器方法post-processors
        exposedObject = initializeBean(beanName,exposedObject,mbd);
      }
    }catch(Throwable ex){
      ...
    }

    // 与singleton缓存有关的操作
    ... 

    return exposedObject;
  }
}

protected Object initializeBean(final String beanName,final Object bean,RootBeanDefinition mbd){
  // BeanNameAware,BeanClassLoaderAware,BeanFactoryAware
  invokeAwareMethods(beanName,bean);

  Object wrappedBean = bean;
  wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean,beanName);

  // afterPropertiesSet,init-method
  invokeInitMethods(beanName,wrappedBean,mbd);

  wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean,beanName);

  return wrappedBean;
}

// 为指定的bean创建一个实例,通过一个恰当的实例化策略:工厂方法,构造函数自动装配，简单实例化(无参实例化)
protected BeanWrapper createBeanInstance(String beanName,RootBeanDefinition mbd,Object[] args){
  Class<?> beanClass = resolveBeanClass(mbd,beanName);

  if(beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()){
    throw new BeanCreationException(mbd.getResourceDescription(),beanName,"Bean class isn't public,and non-public access not allowed");
  }

  // 使用工厂方法创建bean实例
  if(mbd.getFactoryMethodName() != null){
    return instantiateUsingFactoryMethod(beanName,mbd,args);
  }
  ...
  // 使用构造函数自动装配
  if(... || !ObjectUtils.isEmpty(args)){
    return autowireConstructor(beanName,mbd,ctors,args);
  }
  // 没有任何特殊处理,使用无参构造函数实例化
  return instantiateBean(beanName,mbd);
}

// 根据默认构造函数实例化bean
protected BeanWrapper instantiateBean(final String beanName,final RootBeanDefinition mbd){
  // 根据不同策略(使用反射或是cglib进行实例化),如果是反射，则为Constructor.newInstance(...)
  Object beanInstance = getInstantiationStrategy().instantiation(mbd,beanName,this);
  BeanWrapper bw = new BeanWrapper(beanInstance);
  initBeanWrapper(bw);
  return bw;
}

AbstractBeanFactory
// 根据给定的BeanDefinition解析出beanClass
// 过程：将classname解析为一个class对象引用,并且将class对象引用存储到BeanDefinition中
// 返回：返回解析后的beanClass对象
// @param beanName 主要用于错误处理,并不是根据beanName生成beanClass的作用
protected Class<?> resolveBeanClass(final RootBeanDefinition mbd,String beanName,final Class<?>... typesToMatch) throws CannotLoadBeanException {
  try{
    // 确认该BeanDefinition是否指定了beanClass:Object
    // RootBeanDefinition 该BeanDefinition合并了其父类BeanDefinition，继承AbstractBeanDefinition
    // AbstractBeanDefinition
    //    ---- private volatile Object beanClass;//Class对象
    if(mbd.hasBeanClass()){
      return mbd.getBeanClass();
    }
    return doResolveBeanClass(mbd,typeToMatch);
  }
}
```
