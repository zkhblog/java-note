# Spring设计图
![img.png](images/Spring设计图.png)

# 源码分析核心点
① DefaultListableBeanFactory这个工厂实现的具体功能  
② BeanDefinitionRegistry中放入BeanDefinition的过程  

# 注解分析
单例依赖多例，确保多例生效，可以用@Lookup注解，此时不能再用@Autowired。而且单例不能通过@Bean的方式注入容器中，这种方式也是不生效的

# 基础接口
1 Resource + ResourceLoader  
ResourceLoader接口的实现是策略模式的体现。注意ApplicationContext也间接继承了该接口  

### BeanFactory
1 HierarchicalBeanFactory（定义父子工厂）  
    我们常用的ioc容器```AnnotationConfigApplicationContext```，继承于GenericApplicationContext，该容器组合了下面的功能  

2 ListableBeanFactory（能列举所有组件）
    此接口的实现是DefaultListableBeanFactory，可以提供ioc容器中的Bean定义信息相关功能，因为实现了```BeanDefinitonRegistry```  

3 AutowireCapableBeanFactory（提供自动装配共功能）  
    此接口的实现类也是DefaultListableBeanFactory，也是被我们常用的ioc容器所持有  

3 BeanDefinition  

4 BeanDefinitionReader  

5 BeanDefinitionRegistry  

6 ApplicationContext  
ioc容器持有Bean工厂，Bean工厂只是ioc容器的功能之一

7 Aware  
给普通组件装配一些Spring底层组件  
```
自动装配Spring底层组件也可以如下实现  
@Autowired
private ApplicationContext applicationContext;
```

8 DefaultListableBeanFactory（最重要）  
提供访问Bean、BeanDefinition等信息的功能，它被ioc容器所组合  

9 DefaultSingletonBeanRegistry  

10 FactoryBean和Bean  
区别：普通Bean给容器中注册的是该Bean对象，而FactoryBean给容器中注册的是工厂Bean调用getObject()返回的对象，类型是getObjectType()指定的类型  
应用场景：MyBatis和Spring的整合中，所用工厂Bean```SqlSessionFactoryBean```  

11 

# 生命周期中的后置处理器
① BeanFactoryPostProcessor  
② BeanPostProcessor  
③ InitializingBean  
④ BeanDefinitionRegistryPostProcessor  
⑤ ImportBeanDefinitionRegistrar：实现该接口将自定义的组件添加到IOC容器中。
⑥ __outline__beanFactory的三个子接口bean组件实现aware接口，可以装配底层的一些组件，此处利用到的是回调机制，ApplicationContextAwareProcessor的
postProcessBeforeInitialization实现了此机制Bean的功能增强全部是由BeanPostProcessor + InitializaingBean 两个接口完成的FactoryBean和普通Bean的差别，
在遍历所有的beanNames的时候，两者的执行逻辑

# 循环依赖
![img.png](images/循环依赖表述.png)
① Spring是如何解决循环依赖的问题的？  
② 一级可以吗？  
③ 是否可以关闭循环依赖？  
④ 为什么要用三级缓存，用二级缓存可以吗？