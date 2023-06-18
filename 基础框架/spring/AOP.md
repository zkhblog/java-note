# 一 IOC
## 1.1 bean的生命周期

## 1.2 循环依赖
一级缓存：成品对象
二级缓存：半成品对象
三级缓存：lambda表达式

### 1.2.1 三级缓存的泛型研究
ObjectFactory函数式接口的getObject()方法 ：当作一个参数传递到方法中，一般情况下传的是lambda表达式，
可以通过getObject()方法来执行lambda表达式

### 1.2.2 创建流程中的核心步骤
getBean()->doGetBean()->createBean()->doCreateBean()->createBeanInstance()->populateBean()

### 1.2.3 缓存个数
如果只有一个缓存，能不能解决循环依赖：不能区分成品对象和半成品对象。不能解决。  
如果有两个缓存，能不能解决循环依赖：能解决。但是循环依赖过程中包含了代理对象的创建，那么就必须要使用三级缓存了。

三级缓存能解决的原因

# 二 AOP
## 2.1 spring中功能使用
①导包  
spring开发：spring-aspects  
springBoot开发：spring-boot-starter-aop  
②重要步骤  
标识切面类@Aspect  
在切面类的每一个通知方法上标注通知注解，告诉Spring何时何地运行（切入点表达式@Pointcut）  
spring开发要开启基于注解的aop模式：@EnableAspectJAutoProxy  

## 2.2 spring中aop原理分析
[@EnableAspectJAutoProxy注解分析](https://blog.csdn.net/luojinbai/article/details/86670749)

创建流程：  
1）、传入配置类，创建ioc容器  
2）、注册配置类，调用refresh（）刷新容器；  
3）、registerBeanPostProcessors(beanFactory);注册bean的后置处理器来方便拦截bean的创建；  
3.1）、先获取ioc容器已经定义了的需要创建对象的所有BeanPostProcessor  
3.2）、给容器中加别的BeanPostProcessor  
3.3）、优先注册实现了PriorityOrdered接口的BeanPostProcessor；  
3.4）、再给容器中注册实现了Ordered接口的BeanPostProcessor；  
3.5）、注册没实现优先级接口的BeanPostProcessor；  
3.6）、注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，保存在容器中；  
> AOP分析  
> 创建internalAutoProxyCreator的BeanPostProcessor【AnnotationAwareAspectJAutoProxyCreator】  
> 1）、创建Bean的实例  
> 2）、populateBean；给bean的各种属性赋值  
> 3）、initializeBean：  
>   初始化bean的流程：  
>   3.1）、invokeAwareMethods()：处理Aware接口的方法回调  
>   3.2）、applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessBeforeInitialization（）  
>   3.3）、invokeInitMethods()；执行自定义的初始化方法  
>   3.4）、applyBeanPostProcessorsAfterInitialization()；执行后置处理器的postProcessAfterInitialization（）；
> 4）、BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功；--》aspectJAdvisorsBuilder

3.7）、把BeanPostProcessor注册到BeanFactory中；beanFactory.addBeanPostProcessor(postProcessor);  

AOP全称叫做 Aspect Oriented Programming  面向切面编程。它是为解耦而生的，解耦是程序员编码开发过程中一直追求的境界，AOP在业务类的隔离上，绝对是做到了解耦，在这里面有几个核心的概念：
- 切面（Aspect）: 指关注点模块化，这个关注点可能会横切多个对象。事务管理是企业级Java应用中有关横切关注点的例子。 在Spring AOP中，切面可以使用通用类基于模式的方式（schema-based approach）或者在普通类中以`@Aspect`注解（@AspectJ 注解方式）来实现。
- 连接点（Join point）: 在程序执行过程中某个特定的点，例如某个方法调用的时间点或者处理异常的时间点。在Spring AOP中，一个连接点总是代表一个方法的执行。
- 通知（Advice）: 在切面的某个特定的连接点上执行的动作。通知有多种类型，包括“around”, “before” and “after”等等。通知的类型将在后面的章节进行讨论。 许多AOP框架，包括Spring在内，都是以拦截器做通知模型的，并维护着一个以连接点为中心的拦截器链。
- 切点（Pointcut）: 匹配连接点的断言。通知和切点表达式相关联，并在满足这个切点的连接点上运行（例如，当执行某个特定名称的方法时）。切点表达式如何和连接点匹配是AOP的核心：Spring默认使用AspectJ切点语义。
- 引入（Introduction）: 声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被通知的对象上。例如，可以使用引入来使bean实现 `IsModified`接口， 以便简化缓存机制（在AspectJ社区，引入也被称为内部类型声明（inter））。
- 目标对象（Target object）: 被一个或者多个切面所通知的对象。也被称作被通知（advised）对象。既然Spring AOP是通过运行时代理实现的，那么这个对象永远是一个被代理（proxied）的对象。
- AOP代理（AOP proxy）:AOP框架创建的对象，用来实现切面契约（aspect contract）（包括通知方法执行等功能）。在Spring中，AOP代理可以是JDK动态代理或CGLIB代理。
- 织入（Weaving）: 把切面连接到其它的应用程序类型或者对象上，并创建一个被被通知的对象的过程。这个过程可以在编译时（例如使用AspectJ编译器）、类加载时或运行时中完成。 Spring和其他纯Java AOP框架一样，是在运行时完成织入的。
- 这些概念都太学术了，如果更简单的解释呢，其实非常简单：
- 任何一个系统都是由不同的组件组成的，每个组件负责一块特定的功能，当然会存在很多组件是跟业务无关的，例如日志、事务、权限等核心服务组件，这些核心服务组件经常融入到具体的业务逻辑中，如果我们为每一个具体业务逻辑操作都添加这样的代码，很明显代码冗余太多，因此我们需要将这些公共的代码逻辑抽象出来变成一个切面，然后注入到目标对象（具体业务）中去，AOP正是基于这样的一个思路实现的，通过动态代理的方式，将需要注入切面的对象进行代理，在进行调用的时候，将公共的逻辑直接添加进去，而不需要修改原有业务的逻辑代码，只需要在原来的业务逻辑基础之上做一些增强功能即可。
