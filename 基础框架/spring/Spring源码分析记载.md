# 详细源码分析流程图见：[源码分析流程图](https://www.processon.com/)

# 基础接口
1 Resource + ResourceLoader  
ResourceLoader接口的实现是策略模式的体现。注意ApplicationContext也间接继承了该接口  

2 **BeanFactory**  
-- HierarchicalBeanFactory          定义父子工厂  
    我们常用的ioc容器```AnnotationConfigApplicationContext```，继承于GenericApplicationContext，该容器组合了下面的功能  

-- ListableBeanFactory              能列举所有组件
    此接口的实现是DefaultListableBeanFactory，可以提供ioc容器中的Bean定义信息相关功能，因为实现了```BeanDefinitonRegistry```  

-- AutowireCapableBeanFactory       提供自动装配共功能  
    此接口的实现类也是DefaultListableBeanFactory，也是被我们常用的ioc容器所持有  

3 BeanDefinition  

4 BeanDefinitionReader  

5 BeanDefinitionRegistry  

6 ApplicationContext  
ioc容器持有Bean工厂，Bean工厂只是ioc容器的功能之一

7 Aware  
给普通组件装配一些Spring底层组件  
```java
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
1 BeanFactoryPostProcessor  


2 BeanPostProcessor  


3 InitializingBean  

4 BeanDefinitionRegistryPostProcessor  





ImportBeanDefinitionRegistrar：实现该接口将自定义的组件添加到IOC容器中。






















```
/**
 * 此实现类相当于配置了SpringMVC的DispatcherServlet
 * Tomcat启动时就会加载此实现类
 * 1）、创建容器，指定主配置类，此时IOC、AOP等Spring的功能就准备好了
 * 2）、注册了DispatcherServlet，访问Tomcat部署下的应用都会被此类处理，DispatcherServlet就会进入强大的基于注解的mvc处理流程
 *
 */
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {          // Spring传入servletContext

        // Load Spring web application configuration
        // 创建IOC容器
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);                          // 注册主配置类，（用注解版的SpringMVC配置替代配置文件）

        // Create and register the DispatcherServlet
        // 配置了DispatcherServlet，并且保存IOC容器
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);   // 利用servlet规范添加DispatcherServlet
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");                          // 指定好映射
    }
}
```


__outline__beanFactory的三个子接口bean组件实现aware接口，可以装配底层的一些组件，此处利用到的是回调机制，ApplicationContextAwareProcessor的
postProcessBeforeInitialization实现了此机制Bean的功能增强全部是由BeanPostProcessor + InitializaingBean 两个接口完成的FactoryBean和普通Bean的差别，
在遍历所有的beanNames的时候，两者的执行逻辑

