# AOP概念
AOP的作用是在不修改源代码的情况下，可以实现功能的增强

# 创建流程
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


