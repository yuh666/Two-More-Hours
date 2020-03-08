## bean Definition

1. spring framework中定义配置元信息接口,包括以下内容

   ```java
   org.springframework.beans.factory.config.BeanDefinition
   ```

2. bean的类名(getBeanClassName)

3. bean的作用域比如request session,自动绑定的模式比如自动装备等(getScope)

4. 其他bean引用(setDependsOn)

5. 配置设置,比如bean的属性(hasPropertyValues)



## Bean Definition creation

1. 通过BeanDefinitionBuilder.genericBeanDefinition(类名.class)创建BeanDefinitionBuilder,调用addPropertyValue()来设置BeanDefinition的属性,getBeanDefinition()获取BeanDefinition
2. 直接创建GenericBeanDefinition,通过setBeanClass来设置BeanDefinition绑定的class,setPropertyValues()方式来设置属性值



## Bean Definition Registry

1. 注解方式注册Bean,底层都是通过BeanDefinitionRegistry来实现
   - @import
   - @Component
   - @Bean
2. api 配置元信息即BeanDefinitionRegistry,beanDefinition通过上文内容创建
   - 指定bean的名称,beanDefinitionRegistry.registerBeanDefinition(beanName,beanDefinition)
   - 不指定bean的名称即自动生成方式,BeanDefinitionReaderUtils.registryWithGenerateName(beanDefinition,beanDefinitionRegistry)



## SingleBeanRegistry#registrySingleton

1. 外部bean的注册方式,即自己创建bean对象注册bean到DefaultList



## bean instantiation

1. 常规方式:
   - 构造方法
   - 静态工厂方法比如在xml中配置bean的factory-method属性
   - bean工厂方法,定义接口和默认实现,提供创建bean方法,在xml中配置bean指定factory-bean的名称即默认实现和factory-method即接口中Default方法
   - FactoryBean方式来创建,指定接口来实现FactoryBean重写getObject()和getObjectType()
2. 特殊方式:
   - ServiceLoaderFactoryBean中指定serviceType属性,底层等同于通过ServiceLoader读取资源文件方式来获取bean对象在迭代获取资源文件中指定的实现类bean对象,对应的还有ServiceFactoryBean和ServiceListFactoryBean
   - 通过AutowireCapableBeanFactory#createBean的方式来实例化bean对象
   - 通过BeanDefinitionRegistry方式同上文说的BeanDefinitionRegistry内容



## bean init和bean Destroy

1. bean init三种方式,顺序如下
   - @PostConstruct
   - implement initializingBean
   - @bean中指定init-method 
2. bean Destroy三种方式,顺序如下
   - @PreDestroy
   - implement DisposableBean
   - @bean中指定destroy-method



## bean lazy和gc

1. bean lazy:当ApplicationContext.refresh()时不注册懒加载的bean对象,当获取的时候在讲bean实例化,可以解决循环依赖问题,在@autowire中将其他属性bean标记为懒加载
2. spring bean gc步骤:
   - ApplicationContext.close()
   - system.gc()
   - bean 重新finalize方法