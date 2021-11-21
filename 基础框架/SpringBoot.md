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