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

