1 工厂模式，在各种BeanFactory以及ApplicationContext创建中都用到了  
2 模版模式，在各种BeanFactory以及ApplicationContext实现中也都用到了  
3 代理模式，Spring AOP 利用了 AspectJ AOP实现的! AspectJ AOP 的底层用了动态代理  
4 策略模式，加载资源文件的方式，使用了不同的方法，比如：ClassPathResource，FileSystemResource，ServletContextResource，UrlResource但他们都有共同的借口Resource；在Aop的实现中，采用了两种不同的方式，JDK动态代理和CGLIB代理  
5 单例模式，比如在创建bean的时候  
6 观察者模式，spring中的ApplicationEvent，ApplicationListener,ApplicationEventPublisher  
7 适配器模式，MethodBeforeAdviceAdapter,ThrowsAdviceAdapter,AfterReturningAdapter  
8 装饰者模式，源码中类型带Wrapper或者Decorator的都是