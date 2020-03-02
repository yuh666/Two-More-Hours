## 依赖查找

1. 通过类型查找
   - 实时查找:beanFactory.getBean(类.class)
   - 延时查找:beanFactory.getBean(ObjectFactory.class),ObjectFactory通过创建ObjectFactoryCreatingFactoryBean指定targetBeanName为要查找的bean
2. 通过集合类型查找:ListableBeanFactory.getBeansOfType(),返回map中key:bean的名称 value:bean实体对象
3. 通过注解方式查找:ListableBeanFactory.getBeansWithAnnotation()获取有指定注解的bean对象



## 依赖注入

1. 自定义bean对象,比如xml自定义的User对象
2. 容器内创建的bean对象,比如Evroinment
3. 容器内建依赖:比如BeanFactory对象



## BeanFactory和ApplicationContext

1. 依赖注入BeanFactory接口类型的对象是在org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory中指定beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory)为自己,也就是组装后的DefaultListableBeanFactory对象
2. AbstractApplicationContext.getBean()查找bean对象都是查找DefaultListableBeanFactory中对象
3. AbstractApplicationContext和DefaultListableBeanFactory都实现了BeanFactory接口,AbstractApplicationContext的子类AbstractRefreshableApplicationContext中包含了DefaultListableBeanFactory对象;DefaultListableBeanFactory在ApplicationContext创建的时候组装org.springframework.context.support.AbstractRefreshableApplicationContext#refreshBeanFactory,之后再org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory中beanFactory进行封装



## spring ioc的容器

1. beanFactory是spring ioc的底层容器
2. ApplicationContext是BeanFactory的超集,还具有其他特性



## BeanFactory和FactoryBean

1. beanFactory是spring ioc的容器,提供依赖查找和依赖注入,spring通过applicationContext对beanFactory进行在一次的扩展支持更多的功能
2. factoryBean是创建bean的一种方式,内部提供了getObject()方法获取bean对象,getObjectType()bean对象的类型,isSingleTon()是否是单例对象