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

  // 确保RootBeanDefinition中的beanClass已经被解析
  Class<?> resolvedClass = resolveBeanClass(mbd,beanName);
}

doCreateBean... 待续
```