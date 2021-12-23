自动配置原理：  
SpringBootApplication注解是合成注解，其合成注解中的EnableAutoConfiguration注解向容器中导入了**AutoConfigurationImportSelector组件**，EnableAutoConfiguration
注解上标注有AutoConfigurationPackage注解，该注解又给容器中注册了**AutoConfigurationPackages.Registrar.class组件**  
```
AutoConfigurationPackages.Registrar.class  
利用该静态内部类给容器中导入一系列组件，实现是通过获取AutoConfigurationPackage该注解标注的类所在包名，将该包下所有组件导入进容器中，这是包扫描的原理
```

```
@Import(AutoConfigurationImportSelector.class)  
1、利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
3、利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4、从META-INF/spring.factories位置来加载一个文件。
	默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
    spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories
```
虽然自动配置类在项目启动时会全部被加载一遍，但是在某些条件之下才会生效，最终会按需开启自动配置项  
> @ConditionalOnBean：当容器里有指定的bean的条件下。  
> @ConditionalOnMissingBean：当容器里不存在指定bean的条件下。  
> @ConditionalOnClass：当类路径下有指定类的条件下。  
> @ConditionalOnMissingClass：当类路径下不存在指定类的条件下。  
> @ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，
> 代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。




#############################Configuration使用示例######################################################  
属性：proxyBeanMethods：代理bean的方法  
    Full(proxyBeanMethods = true)、【保证每个@Bean方法被调用多少次返回的组件都是单实例的】  
    Lite(proxyBeanMethods = false)【每个@Bean方法被调用多少次返回的组件都是新创建的】  
    组件依赖必须使用Full模式默认。其他默认是否Lite模式(如果组件不被其他组件依赖，使用lite模式可以加快容器启动速度，
    因为无需判断容器中是否有该组件，调用方法即创建组件)  
    
总结：  
①SpringBoot先加载所有的自动配置类 xxxxxAutoConfiguration
②每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定
③生效的配置类就会给容器中装配很多组件
④只要容器中有这些组件，相当于这些功能就有了
⑤定制化配置
  5.1 用户直接自己@Bean替换底层的组件
  5.2 用户去看这个组件是获取的配置文件什么值就去修改。
xxxxxAutoConfiguration ---> 组件  ---> xxxxProperties里面拿值  ----> application.properties