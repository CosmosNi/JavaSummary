# Spring总结

###### 1.1 核心类DefaultListableBeanFatory

DefaultListableBeanFatory是整个bean加载的核心部分，是spring注册及加载bean的默认实现。以下即DefaultListableBeanFatory的组成结构。

- AliasRegistry：定义对alias的简单增删改等操作。

- SimpleAliasRegister：使用map作为alias的缓存，并对接口AliasRegistry进行实现。

- SingletonBeanRegistry：定义对单例的注册及获取。

- BeanFactory：定义获取bean及bean的各种属性。

- DefaultSingletonBeanRegistry：对接口SingletonBeanRegistry各函数实现。

- HierarchicalBeanFactory：继承BeanFactory，在BeanFactory基础上增加对parentFactory的支持。

- BeanDefinitionRegistry：定义对BeanDefinition的增删改操作。

- FactoryBeanRegistrySupport：在DefaultSingletonBeanRegistry的基础上增加对FactoryBean的特殊处理功能。

- ConfigurableBeanFactory：提供配置Facory的各种方法。

- ListableBeanFactory：根据各种条件获取bean的配置清单。

- AbstractBeanFactory：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。

- AutowireCapableBeanFactory：提供创建bean、自动注入、初始化以及应用bean的后处理器。

- AbstractAutowireCapableBeanFactory：综合AbstractBeanFactory并对接口AutowireCapableBeanFactory进行实现。

- ConfigurableListableBeanFactory：BeanFactory配置清单，指定忽略类型及接口等。

  

###### 1.2  XmlBeanDefinitionReader (xml文件的读取)

- ResourceLoader：定义资源加载器，主要应用于根据给定的资源文件地址返回对应的resource。
- BeanDefinitionReader：主要定义资源文件读取并转换为BeanDefinition的各个功能。
- EnvironmentCapable：获取Environment方法
- DocumentLoader：定义从资源文件加载到转换为Document的功能
- AbstractBeanDefinitionReader：对EnvironmentCapable、BeanDefinitionReader的功能进行实现。
- BeanDefinitionDocumentReader：定义读取Document并注册BeanDefinition功能。
- BeanDefinitionParserDelegate：定义解析Element的各种方法。



###### 1.3 spring对于循环依赖的解决

- 构造器循环依赖：表示通过构造器注入构成的循环依赖。此依赖是无法解决的，只能抛出BeanCurrentlyInCreationException
- setter循环依赖：表示通过setter注入方式构成的循环依赖。对于setter注入造成的依赖是通过Spring容器提前暴露刚完成构造注入但未完成其他步骤（如setter注入）的bean来完成的，而且只能解决单例作用域的bean循环依赖。通过提前暴露一个单例工厂方法，从而使其他的bean能引用到该bean。通过“setAllowCircularReferences（false）”来禁用循环依赖
- prototype范围的依赖处理：对于“prototype”作用域bean，spring容器无法完成依赖注入，因为spring容器不进行缓存“prototype”作用域的bean，因此无法提前暴露一个创建中的bean。



###### 1.4 spring生命周期

- 容器启动后，对bean进行初始化。
- 按照的bean的定义，注入属性
- 检测该对象是否实现了Aware接口，并将Aware实例注入到bean中
- 实现BeanPostProcessor接口，进行一些自定义的前置方法处理。
- 调用自定义的初始化方法。如postConstruct，init-method等等
- 实现BeanPostProcessor接口，进行一些自定义的后置方法处理。
- 容器关闭后，如果bean实现了DisposableBean，则会回调该接口的destroy（）方法
- 通过给destroy-method指定函数，可以在bean销毁前执行指定的逻辑

###### 1.5 bean的作用域

1. ingleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。
2. prototype：为每一个bean请求提供一个实例。
3. request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
4. session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
5. global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。

