# spring钩子

###### 1.1 Aware接口族

spring提供了各种Aware接口，方便从上下文中获取当前的运行环境。常见的有如下几个：

- BeanFactoryAware
- BeanNameAware
- ApplicationContextAware
- EnvironmentAware
- BeanClassLoaderAware

###### 1.2 InitializingBean接口和DisposableBean接口

- InitializingBean：当一个Bean实现InitializingBean，#afterPropertiesSet方法里面可以添加自定义的初始化方法或者做一些资源初始化操作
- DisposableBean接口只有一个方法#destroy，作用是：当一个单例Bean实现DisposableBean，#destroy可以添加自定义的一些销毁方法或者资源释放操作



###### 1.3 ImportBeanDefinitionRegistrar接口

###### 1.4 BeanPostProcessor接口和BeanFactoryPostProcessor接口

Spring的Bean后置处理器接口,作用是为Bean的初始化前后提供可扩展的空间。BeanFactoryPostProcessor可以对bean的定义（配置元数据）进行处理。也就是说，Spring IoC容器允许BeanFactoryPostProcessor在容器实际实例化任何其它的bean之前读取配置元数据，并有可能修改它。实现BeanPostProcessor接口可以在Bean(实例化之后)初始化的前后做一些自定义的操作，但是拿到的参数只有BeanDefinition实例和BeanDefinition的名称，也就是无法修改BeanDefinition元数据,这里说的Bean的初始化是： 1）bean实现了InitializingBean接口，对应的方法为afterPropertiesSet 2）在bean定义的时候，通过init-method设置的方法 PS:BeanFactoryPostProcessor回调会先于BeanPostProcessor 实现BeanPostProcessor接口可以在Bean(实例化之后)初始化的前后做一些自定义的操作，但是拿到的参数只有BeanDefinition实例和BeanDefinition的名称，也就是无法修改BeanDefinition元数据,这里说的Bean的初始化是： 1）bean实现了InitializingBean接口，对应的方法为afterPropertiesSet 2）在bean定义的时候，通过init-method设置的方法 PS:BeanFactoryPostProcessor回调会先于BeanPostProcessor



###### 1.5 BeanDefinitionRegistryPostProcessor/BeanDefinitionRegistryPostProcessor

可以看作是BeanFactoryPostProcessor和ImportBeanDefinitionRegistrar的功能集合，既可以获取和修改BeanDefinition的元数据，也可以实现BeanDefinition的注册、移除等操作。



###### 1.6 FactoryBean

一般情况下，Spring通过反射机制利用bean的class属性指定实现类来实例化bean ，实例化bean过程比较复杂。FactoryBean接口就是为了简化此过程，把bean的实例化定制逻辑下发给使用者。



###### 1.7 ApplicationListener

ApplicationListener是一个接口，里面只有一个onApplicationEvent(E event)方法，这个泛型E必须是ApplicationEvent的子类，而ApplicationEvent是Spring定义的事件，继承于EventObject，构造要求必须传入一个Object类型的source，这个source可以作为一个存储对象。 将会在ApplicationListener的onApplicationEvent里面得到回调。如果在上下文中部署一个实现了ApplicationListener接口的bean，那么每当在一个ApplicationEvent发布到 ApplicationContext时，这个bean得到通知。